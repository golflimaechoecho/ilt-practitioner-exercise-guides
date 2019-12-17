# Lab 3.2: Create a module

We’re going to start off with a relatively simple `review` module that should
manage a named user and some configuration files. We'll also manage a
file in a separate class.

First, we will add a single `$user` parameter to define the username to manage. If
the user is `root`, the path to the user’s home directory will have to change
to reflect that. Then we’ll add a class that will manage the `/etc/motd`.

Since the catalog is compiled on the master, we'll need to use some logic to
extrapolate the user’s home directory.

```ruby
$homedir = $user ? {
  'root'  => '/root',
  default => "/home/${user}",
}
```

For the rest of this course, we will refer to the `/etc/puppetlabs/code/environments/production/modules` as `[modulepath]`. You can find this again with running `cd $(puppet agent --configprint environmentpath)/production/modules` and will be used for all labs.

We haven't explained to you how to do most of these tasks, so please request
assistance from the instructor as needed.

# Steps

### Create the review module using PDK.

1. Change directory to your `[modulepath]` by running
  
  ```$ cd $(puppet agent --configprint environmentpath)/production/modules```

1. Run this command:

  ```$ pdk new module```

1. You will see several questions requiring an answer. Enter these answers:

  **_Replace the N in studentN with your student number (for example, `student8`)._**

  | Question           | Answer              |
  | ------------------ |:-------------------:|
  | Module Name        | `review`            |
  | Forge Name         | `studentN`          |
  | Credit author      | `Student N`         |
  | License            | `Apache-2.0`        |
  | Operating reviews  | RedHat              |

### Update the class

1. Navigate to your `review` directory.
  ```cd review```
1. Create the class.
  ```pdk new class review```
1. Edit `manifests/init.pp`
    * Add a single parameter, `$user`, defaulting to `review`
1. The class should manage:
    * The `$user` user with a `.bashrc` in their home directory.
    * Ensure that the `puppet` service is stopped.
    * Declare a class named `review::motd`.

### Create a new class

1. Create a class named `review::motd` in the proper location.

  ```pdk new class motd```

1. Leave the body of the class blank. We'll finish it soon.

### Test your work

**_Expect an error from the following command._**

1. Create your smoke test file to validate your work.
  ```mkdir examples```
1. Edit `examples/init.pp`
1. Validate your syntax using `pdk`.
  ```pdk validate```
1. Using puppet apply, validate your smoke test.
  ```puppet apply examples/init.pp --noop```

# Solution

### Your module structure should resemble

[root@training modules]# tree review/

```shell
  review/
  ├── examples
  │   ├── motd.pp
  │   └── init.pp
  ├── files
  │   └── bashrc
  └── manifests
      ├── motd.pp
      └── init.pp
```

#### Example file: `review/manifests/motd.pp`

```ruby
class review::motd {
}
```

#### Example file: `review/manifests/init.pp`

```ruby
# A description of what this class does
#
# @summary A short summary of the purpose of this class
#
# @example
#   include review
class review (
  $user = 'review',
) {
  include review::motd

  $homedir = $user ? {
    'root'  => '/root',
    default => "/home/${user}",
  }

  user { $user:
    ensure     => present,
    shell      => '/bin/bash',
    managehome => true,
  }

  file { "${homedir}/.bashrc":
    ensure => file,
    owner  => $user,
    group  => $user,
    mode   => '0644',
    source => 'puppet:///modules/review/bashrc'
  }

  service { 'puppet':
    ensure => stopped,
    enable => false,
  }
}
```

#### Example file: `review/files/bashrc`

```bash
# .bashrc
# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi
```

|  [Previous Lab](../lab-03.1-Explore-classification)  |  [Next Lab](../lab-03.3-Manage-a-file)  |
