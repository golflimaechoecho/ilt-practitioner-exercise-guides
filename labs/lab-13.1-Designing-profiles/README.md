# Lab 13.1: Designing profiles

It is useful to provide a concise definition of the various technology stacks used in an infrastructure. For example, we could define a WordPress stack as including Apache, MySQL, and PHP bindings for both. Or we could define another implementation of a WordPress stack as including Nginx, PostgreSQL, and PHP bindings. Similarly, we could define a Splunk client stack as including the Splunk agent, configured to talk to our central Splunk server.

Each of these technology stacks, or profiles, can be combined as abstracted building blocks to construct the full configuration of a node.

This lab doesn't expect you to write any real code. Instead, you should just design the appropriate abstraction layers without concerning yourself with implementation details of each module.

For example, a WordPress profile might look something like the code below. In order to configure a server to host WordPress, we need the requirements for WordPress to be met. A quick web search and some research indicates that WordPress is a PHP-powered application that requires a webserver, a MySQL database, and PHP. Further, for PHP-based web apps to work, the appropriate bindings, libraries or modules should be installed and enabled for the webserver and the database server.

Note that the class doesn't actually work as written, but it gets the point across and we can flesh out details later.

```ruby
class profile::wordpress {
  include apache
  include apache::mod::php
  include mysql::server
  include mysql::php

  class { '::wordpress':
    install_dir => '/var/www/html',
    require     => Class['apache'],
  }
}
```

Continue on from this example and add profiles for

* An Exim email server
* A Nagios system monitor
* A Splunk log monitor.

For each of the above, you will need to research what it takes to set up a server. A mail server stack, for example, would need an Mail Transfer Agent (MTA) and a Mail Delivery Agent (MDA), both of which are provided by Exim. In addition, you will need some content filters such as ClamAV, Spamassassin etc.

Do some basic research, or otherwise gather a list of components that you will need to manage to realize a solution for the above. Describe a `profile` class as a set of declared classes. The content of those classes can be fleshed-out later as needed.

Component modules that you might use could include `mysql`, `apache`, `php`, `exim`, `nagios`, `splunk`, `spamd`, `clamav`, etc. or you could design stacks with other components to meet your needs.

## Steps:

1. If it doesn't exist, create a new `profile` directory.
    * `cd [control-repo]/site/`
    * `mkdir -p profile/manifests`
1. Create a class in pseudocode or abbreviated code for each profile required.
    * Edit `[modulepath]/profile/manifests/splunk.pp`
    * Edit `[modulepath]/profile/manifests/nagios.pp`
    * etc.
1. Do not validate and test your classes as they're not intended to be complete
   working solutions.

#### Discussion Questions

* Can you quickly identify unique profiles that fit your own infrastructure?
* Should profiles take parameters? Why or why not?
* What might you do when you have a profile that needs to be customized?

# Solution

### Your module structure should resemble

```
[root@training site]# tree profile/
profile/
└── manifests
    ├── exim.pp
    ├── nagios.pp
    ├── splunk
    │   └── server.pp
    └── splunk.pp
```

#### Example file: `profile/manifests/nagios.pp`

```ruby
class profile::nagios {
  include ::nagios
}
```

#### Example file: `profile/manifests/exim.pp`

```ruby
class profile::exim {
  include clamav
  include spamd
  class { '::exim':
    use_smarthost        => true,
    smarthost_route_data => 'mta.example.com',
    spam_scan            => true,
    spamd_address        => '127.0.0.1 783',
  }
}
```

#### Example file: `profile/manifests/splunk.pp`

```ruby
class profile::splunk {
  class { '::splunk':
    logging_server  => lookup('splunk_server'),
  }
}
```

#### Example file: `profile/manifests/splunk/server.pp`

```ruby
class profile::splunk::server {
  class { 'splunk':
    deploy            => 'server',
    splunk_admin      => 'harrison',
    splunk_admin_pass => 'wooki3',
  }
}
```

|  [Previous Lab](../lab-11.1-Configure-hiera)  |  [Next Lab](../lab-13.2-Designing-roles)  |
