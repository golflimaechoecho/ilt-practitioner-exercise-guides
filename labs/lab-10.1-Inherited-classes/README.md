# Lab 10.1: Inherited classes

Many applications share a common core configuration. One way to limit the amount of code duplication when designing for this is to create a base class and inherit its functionality in more specific classes. Let's assume for the purposes of this class that we have a series of web applications that all share the same LAMP stack. A main `webapp` class that reflects this might look like:

```ruby
class webapp {
  include mysql::server

  class { 'mysql::bindings':
    php_enable => true,
  }

  include apache
  include apache::mod::php

  apache::vhost { $facts['fqdn']:
    priority   => '10',
    vhost_name => $facts['fqdn'],
    port       => '80',
    docroot    => '/var/www/html',
  }

  @@haproxy::balancermember { $facts['fqdn']:
    listening_service => 'webapp',
    ports             => '80',
  }
}
```

If we wanted to manage a WordPress web application, we could do that with a `webapp::wordpress` subclass that inherits the `webapp` functionality and also declares the `::wordpress` module. This example also overrides the load balancer configuration and the `docroot`.

```ruby
class webapp::wordpress inherits webapp {
  include wordpress

  # override load balancer to 'wordpress' listening service
  Haproxy::Balancermember[$facts['fqdn']] {
    listening_service => 'wordpress',
  }
}
```

**_Older versions of Puppet resolved class names like variables, so `wordpress` would resolve to local scope first, or `webapp::wordpress`. Since that class exists, it essentially tried to include itself. This is no longer an issue in Puppet 4.x. This is why you'll see the common pattern of including classes explicitly as top-scope, e.g. `include ::wordpress`._**

Your job is to refactor this example to use composition rather than inheritance. To do this, you'll need to pass in parameters to the base class.

**_You may encounter a duplicate resource error if your node group is still classified with the `ordering` class. Remove that classification to proceed._**

## Steps:

### Obtain the component modules

1. Install the modules used for this lab:
  * Edit `[control-repo]/Puppetfile`.
  * Uncomment the following lines in the Puppetfile.  
      
      ```
      mod 'puppetlabs/apache'
      mod 'puppetlabs/haproxy'
      mod 'hunner/wordpress'
      ```

1. Read the documentation for these modules as needed.
    * You may use the README file in the module directory or the Puppet Forge
      module page as you prefer.

### Create a new module `webapp` with pdk

1. Change directory to your `[modulepath]`  

    ```$ cd $(puppet agent --configprint environmentpath)/production/modules```

1. Create the module structure

    ```$ pdk new module```

1. Press `Enter`. You will see several questions requiring an answer. Enter the answers as you see below:

    **_Replace the N in studentN with your student number, e.g. `student8`_**

    | Question           | Answer              |
    | ------------------ |:-------------------:|
    | Module Name        | `webapp`            |
    | Forge Name         | `studentN`          |
    | Credit author      | `Student N`         |
    | License            | `Apache-2.0`        |
    | Operating systems  | RedHat              |

1. Create the base `webapp` class.

    ```pdk new class webapp```

1. Edit `manifests/init.pp`
    * Accept a String `$docroot`, defaulting to `/var/www/html`
        * Pass `$docroot` to the `apache::vhost` declaration
    * Accept a String `$app_name`, defaulting to `webapp`
        * Pass the `$app_name` to the `haproxy::balancermember` declaration.
1. Create a `webapp::wordpress` class.

    ```pdk new class wordpress```

1. Edit `manifests/wordpress.pp`
    * Declare the `webapp` class and pass parameters as needed.

### Test your module

1. Validate and test your classes.

    ```pdk validate```

1. Edit `examples/wordpress.pp`
1. Run a local puppet apply

    ```puppet apply webapp/examples/wordpress.pp```

    The output should be similar to this:

    ```
    [root@training modules]# puppet apply webapp/examples/init.pp
    notice: /Stage[main]/Mysql::Server/Package[mysql-server]/ensure: created
    notice: /Stage[main]/Mysql::Server/Service[mysqld]/ensure: ensure changed
    'stopped' to 'running'
    notice: /Stage[main]/Website/Apache::Vhost[training.puppetlabs.vm]/File[/var
    /log/httpd]/ensure: created
    notice: /Stage[main]/Apache/Package[httpd]/ensure: created
    info: /Stage[main]/Apache/Package[httpd]: Scheduling refresh of Service[httpd]
    [...]
    notice: /Stage[main]/Wordpress::Db/Database[wordpress]/ensure: created
    notice: /Stage[main]/Wordpress::Db/Database_user[wordpress@localhost]/ensure:
    created
    notice: /Stage[main]/Wordpress::Db/Database_grant[wordpress@localhost/wordpress]
    /privileges: privileges changed '' to 'all'
    notice: /Stage[main]/Apache/Service[httpd]/ensure: ensure changed 'stopped' to
    'running'
    notice: /Stage[main]/Apache/Service[httpd]: Triggered 'refresh' from 53 events
    notice: Finished catalog run in 29.09 seconds
    ```

## Extra Credit

While we're at it, you can earn some brownie points by extending the functionality of the module to support multiple Linux distributions using the `params` pattern. Calculate the default path of the `$docroot` in `params.pp` and inherit that class in `webapp` to make the variables available.

1. Create a `webapp::params` class that calculates the default `$docroot` based on the `$facts['os']['family']` fact
    
    ```pdk new class params```

1. Edit `manifests/params.pp`
1. Refactor `webapp` to inherit from the `webapp::params` class and use it to calculate appropriate platform defaults
   * Edit `manifests/init.pp`

# Solution

### Your module structure should resemble

```
[root@training modules]# tree webapp/
webapp/
├── examples
│   ├── init.pp
└── manifests
    ├── init.pp
    ├── params.pp
    └── wordpress.pp
```

#### Example file: `webapp/manifests/wordpress.pp`

```ruby
class webapp::wordpress {
  include wordpress

  class {'webapp':
    docroot => '/opt/wordpress',
    app_name => 'wordpress',
  }
}
```

#### Example file: `webapp/manifests/init.pp`

```ruby
class webapp (
  String $docroot = '/var/www/html',
  String $app_name = 'webapp'
) {
  include mysql::server
  class { 'mysql::bindings':
    php_enable => true,
  }

  include apache
  include apache::mod::php
  apache::vhost { $facts['fqdn']:
    priority   => '10',
    vhost_name => $facts['fqdn'],
    port       => '80',
    docroot    =>  $docroot,
  }

  @@haproxy::balancermember { $fqdn:
    listening_service => $app_name,
    ports             => '80',
  }
}
```

### Extra Credit

#### Example file: `webapp/manifests/init.pp`

```ruby
class webapp (
  String $docroot,
  String $app_name = 'webapp'
) inherits webapp::params {
  include mysql::server
  class { 'mysql::bindings':
    php_enable => true,
  }

  include apache
  include apache::mod::php
  apache::vhost { $facts['fqdn']:
    priority   => '10',
    vhost_name => $facts['fqdn'],
    port       => '80',
    docroot    =>  $docroot,
  }

  @@haproxy::balancermember { $fqdn:
    listening_service => $app_name,
    ports             => '80',
  }
}
```

#### Example file: `webapp/manifests/params.pp`

```ruby
class webapp::params {
  case $facts['os']['family'] {
    'RedHat': {
      $docroot = '/var/www/html'
    }
    'Debian': {
      $docroot = '/var/www'
    }
    default: {
      fail("Module ${module_name} is not supported on ${facts['os']['family']}")
    }
  }
}
```

|  [Previous Lab](../lab-09.2-Using-augeas)  | [Next Lab](../lab-11.1-Configure-hiera)  |
