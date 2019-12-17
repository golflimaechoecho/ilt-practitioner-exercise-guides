# Lab 6.2 Iterating with each:

Often codebases that grow organically end up with snippets of cut-and-pasted repeated code. Some parts of the repetition will be identical, and some parts will have subtle differences. Since this eventually becomes an unmaintainable mess, we must proactively refactor once we start down this path.

Take this sample class below and refactor it to reduce the spaghetti creep. Refactoring commonalities into named variables provides context and makes it easier to update and/or refactor later.

```ruby
class system::admins {
  require mysql::server
  mysql_user { 'zack@localhost':
    ensure               => present,
    max_queries_per_hour => 1200,
  }
  mysql_user { 'monica@localhost':
    ensure               => present,
    max_queries_per_hour => 600,
  }
  mysql_user { 'ralph@localhost':
    ensure               => absent,
  }
  mysql_user { 'brad@localhost':
    ensure               => present,
    max_queries_per_hour => 600,
  }
  mysql_user { 'luke@localhost':
    ensure               => present,
    max_queries_per_hour => 600,
  }
  user { ['zack', 'monica', 'ralph', 'brad', 'luke']:
    ensure => present,
  }
}
```

Let's use Puppet 6 iteration. What commonalities can you see in this code? What differences do you see? Have you spotted the bug?

### Create a new `admins` class

1. Change directory to your `[modulepath]/system`

    ```$ cd $(puppet agent --configprint environmentpath)/production/modules/system```

1. Make a new class `admins`.

    ```$ pdk new class admins```

1. Make a list of all admin users that exist.
  * Make a list of all users who have had their accounts retired.
  * Identify the parameter(s) common to most users and the outlier(s).
  * Correct any errors discovered.
  * Turn each list into the appropriate data structure.
    * You may include parameters for each user, or rely on resource defaults.
    * Edit `manifests/admins.pp`
  * Write the correct lambda block to manage resources for each user.
    * Manage a `mysql_user` resource.
    * Manage a `user` resource.

### Test and enforce your code

1. Install the `puppetlabs/mysql` module if needed.

    ```$ puppet module install puppetlabs/mysql```

1. Validate and test your new class.

    ```$ pdk validate```

1. Test your new class.

    ```$ puppet apply -e 'include system::admins'```

1. Commit and push your changes.

### Extra credit

The [`pick()` function](https://forge.puppet.com/puppetlabs/stdlib#pick) can be used to select the first non-undefined value from a list of values. It could be useful when handling default values in your refactored class.

**Example:**

```ruby
$default_udlc_stratum = 15

# $udlc_stratum has been set elsewhere and is either
# "undef" or a numeric value

class { 'ntp':
  udlc_stratum => pick($udlc_stratum, $default_udlc_stratum),
}
```

Refactor your code with the `pick()` function to remove even more duplication.

# Solution

### Your module structure should resemble

```
[root@training modules]# tree system/
system/
├── examples
│   └── admins.pp
└── manifests
    └── admins.pp
    └── admins_pick.pp  # this file will only exist if you did the extra credit
```

#### Example file: `system/manifests/admins.pp`

```ruby
class system::admins {
  require mysql::server
# You can either use this resource default, or declare each parameter
# directly in the $admins hash
#   Mysql_user {
#     max_queries_per_hour => '600',
#   }

  $retired = [ 'ralph' ]
  $admins  = {
    'brad'   => { max_queries_per_hour =>  '600' },
    'monica' => { max_queries_per_hour =>  '600' },
    'luke'   => { max_queries_per_hour =>  '600' },
    'zack'   => { max_queries_per_hour => '1200' },
  }

  $admins.each |$user, $params| {
    mysql_user { "${user}@localhost":
      ensure               => present,
      max_queries_per_hour => $params['max_queries_per_hour'],
    }

    user { $user:
      ensure     => present,
      managehome => true,
    }
  }

  $retired.each |$user| {
    mysql_user { "${user}@localhost":
      ensure => absent,
    }

    user { $user:
      ensure => absent,
    }
  }
}
```

#### Example file: `system/examples/admins.pp`

```ruby
include system::admins
```

### Extra credit

#### Example file: `system/manifests/admins_pick.pp`

```ruby
class system::admins_pick {
  require mysql::server

  $default_max_queries_per_hour = '600'

  $retired = [ 'ralph' ]
  $admins  = {
    'brad'   => {},
    'monica' => {},
    'luke'   => {},
    'zack'   => { max_queries_per_hour => '1200' },
  }

  $admins.each |$user, $params| {
    mysql_user { "${user}@localhost":
      ensure               => present,
      max_queries_per_hour => pick($params['max_queries_per_hour'],
                                   $default_max_queries_per_hour),
    }

    user { $user:
      ensure     => present,
      managehome => true,
    }
  }

  $retired.each |$user| {
    mysql_user { "${user}@localhost":
      ensure => absent,
    }

    user { $user:
      ensure => absent,
    }
  }
}
```

|  [Previous Lab](../lab-06.1-Validating-parameters)  |  [Next Lab](../lab-07.1-Ordering-methods)  |