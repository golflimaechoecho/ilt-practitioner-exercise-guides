# Lab 7.1: Ordering methods

We are going to add relationships between resources and classes to help Puppet order the enforcement of our catalog.

In order to allow the idempotent behavior of the `include()` function, classes do not follow the standard rules of containment. Our first task is to write a simple MySQL wrapper class that will use the functionality of the `puppetlabs/mysql` module to manage the configuration of MySQL on our nodes. In order for other classes to be able to set relationships on our class, we need to contain the `mysql*` classes that we use. Your class should manage the MySQL server with a root database user password and bindings for `perl` and `php`. Manually remove the packages that are already installed so you can see the class being enforced.

This class might start out like below. Add any containment needed.

```ruby
class ordering::mysql {
  # How do we make sure these classes don't float off the relationship graph?
  class { '::mysql::server':
    root_password => 'strongpassword',
  }

  class { '::mysql::bindings':
    php_enable  => true,
    perl_enable => true,
  }
}
```

Include a `notify` resource in the main `ordering` class with relationships set so you can see whether you've contained the MySQL classes properly.

```ruby
notify { 'This should come after the entire MySQL class is enforced':
  require => Class['ordering::mysql'],
}
```

### Prerequisites

1. Install the MySQL module if needed (it should be installed from the last lab).

    ```puppet module install puppetlabs/mysql```

1. Uncomment the following reference in your `[control-repo]/Puppetfile`.

    `mod 'puppetlabs/mysql'`

1. Commit and push your control repo changes.

## Create a new module

1. Change directory to your `[modulepath]`.

    ```$ cd $(puppet agent --configprint environmentpath)/production/modules```
  
1. Create a new module called `ordering`.

    ```$ pdk new module```

1. You will see several questions requiring an answer. Enter the answers as you see below:

    **_Replace the N in studentN with your student number, e.g. `student8`_**

    | Question           | Answer              |
    | ------------------ |:-------------------:|
    | Module Name        | `ordering`          |
    | Forge Name         | `studentN`          |
    | Credit author      | `Student N`         |
    | License            | `Apache-2.0`        |
    | Operating systems  | RedHat              |

1. Change directory to your `[modulepath]/ordering`

    ```$ cd ordering```

1. Create a new class `ordering`.

    ```$ pdk new class ordering```

1. Edit `manifests/init.pp`
1. Create the `mysql` class.

    ```$ pdk new class mysql```

1. Edit `manifests/mysql.pp`

### Validate and commit your code

1. Validate and test the new class:

    ```$ pdk validate```

1. Commit your changes to the repo.
    * Init your module: `git init`
    * Add the origin: `git remote add origin git@gitlab.classroom.puppet.com:puppet/ordering.git`
    * Checkout the branch: `git checkout -b studentN`
    * Add the files: `git add --all`
    * Create a commit: `git commit -m "Initial Commit"`
    * Push the commit to the Git server: `git push origin studentN`

1. Uncomment the following entry in the `[control-repo]/Puppetfile`.
    **_Replace the “N” in these answers with your student number (for example, student8)._**

    ```
    mod 'ordering',
      :git    => 'git@gitlab.classroom.puppet.com:puppet/ordering.git',
      :branch => 'studentN'
    ```

1. Commit and push your control repo changes.
1. Run `puppet agent -t`.

#### Expected output

Pay close attention to the order in which resources were enforced.
**_Your output should be similar to this_**

```
$ puppet agent -t
Notice: Local environment: 'production' doesn't match server specified node environment 'training', switching agent to 'training'.
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Loading facts
Info: Caching catalog for training.puppetlabs.vm
Info: Applying configuration version 'b76bffe'
Notice: Hello, my name is training
Notice: /Stage[main]/Profile::Base/Notify[Hello, my name is training]/message: defined 'message' as 'Hello, my name is training'
Notice: /Stage[main]/Mysql::Server::Config/File[mysql-config-file]/ensure: defined content as '{md5}4b16ed3375eaa96a2bc1b7aa00c5dd46'
Notice: /Stage[main]/Mysql::Server::Install/Package[mysql-server]/ensure: created
Notice: /Stage[main]/Mysql::Server::Service/Service[mysqld]/ensure: ensure changed 'stopped' to 'running'
Info: /Stage[main]/Mysql::Server::Service/Service[mysqld]: Unscheduling refresh on Service[mysqld]
Notice: This should come after the entire MySQL class is enforced
Notice: /Stage[main]/Ordering/Notify[This should come after the entire MySQL class is enforced]/message: defined 'message' as 'This should come after the entire MySQL class is enforced'
Notice: Applied catalog in 15.66 seconds
```

# Solution

### Your module structure should resemble

```
[root@training modules]# tree ordering/
ordering/
├── examples
|   ├── init.pp
|   └── mysql.pp
└── manifests
    ├── init.pp
    └── mysql.pp
```

#### Example file: `ordering/examples/init.pp`

```ruby
include ordering
```

#### Example file: `ordering/manifests/init.pp`

```ruby
class ordering {
  include ordering::mysql

  notify { 'This should come after the entire MySQL class is enforced':
    require => Class['ordering::mysql'],
  }
}
```

#### Example file: `ordering/manifests/mysql.pp`

```ruby
class ordering::mysql {
  class { '::mysql::server':
    root_password    => 'strongpassword',
  }

  class { '::mysql::bindings':
    php_enable  => true,
    perl_enable => true,
  }

  contain mysql::bindings
  contain mysql::server
}
```

|  [Previous Lab](../lab-06.2-Iterating-with-each)  |  [Next Lab](../lab-07.2-Export-a-resource)  |
