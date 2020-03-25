# Lab 5.2: Defined type

You may be familiar with the `/etc/skel` directory, which has content that  should pre-populate the home directory of new Linux users. Using defined types, we can accomplish a similar effect completely within Puppet.  What's more, the user definitions can update to match any configuration changes we make, and can be enforced consistently across platforms.

In this exercise, you will create a simple defined type that wraps a user type and a file type to manage a user and assets. You may manage any other assets you like. For example, you may manage a group for your users or an SSH key, or anything else that sounds interesting.

**_You can generate password hashes to use in Puppet code using the standard OpenSSL tool:  `openssl passwd -1`._**

## Create a defined type

1. Create a defined type to describe the related resources that make up a `system::managed_user`. Make sure you are still in the `[modulepath]/system` directory.

    ```pdk new defined_type managed_user```

1. Edit `manifests/managed_user.pp`
    * Add a `password` parameter and pass its value to the `user` type.
    * Manage the `.bashrc` file when enforced on Linux.
        * The `$kernel` or `$facts['os']['family']` fact might be useful.
        * Print a message with `echo` or generate a funny message with `cowsay`.
        * If you need a starting point, try `cat ~/.bashrc` on your VM.
1. Validate and test your type.
    * Create the `examples` directory.

      ```mkdir examples```

    * Edit `examples/managed_user.pp` and declare at least two `system::managed_user` resources.

      **Example declarations**

      ```ruby
      $password = '$1$HdDw//gC$2VBiQ1x5blLPwNS.G.Iw21'

      system::managed_user { ['aaron', 'kaitlin', 'jose']:
        password => $password,
      }
      ```

      ```pdk validate```

      ```puppet apply examples/managed_user.pp --noop```

      ```puppet apply examples/managed_user.pp```

      The output should be similar to this

      ```plaintext
      [root@1970nix0 ~]# puppet apply examples/managed_user.pp
      Info: Using configured environment 'production'
      Info: Retrieving pluginfacts
      Info: Retrieving plugin
      Info: Loading facts
      Info: Caching catalog for master.puppetlabs.vm
      Info: Applying configuration version '1464215161'
      Notice: /Stage[main]/System::Managed_user[jose]/User[jose]/ensure: created
      Notice: /Stage[main]/System::Managed_user[jose]/File[/home/jose]/group: group changed 'jose' to 'wheel'
      Notice: /Stage[main]/System::Managed_user[jose]/File[/home/jose]/mode: mode changed '0700' to '0755'
      Notice: /Stage[main]/System::Managed_user[jose]/File[/home/jose/.bashrc]/content:
      --- /home/jose/.bashrc    2015-11-20 05:02:30.000000000 +0000
      +++ /tmp/puppet-file20160525-62457-xtsnxo 2016-05-25 22:26:18.496743808 +0000
      @@ -1,11 +1,10 @@
      -# .bashrc
      -
       # Source global definitions
       - if [ -f /etc/bashrc ]; then
       + [[ -f /etc/bashrc ]] && source /etc/bashrc
      [...]
      Notice: /Stage[main]/System::Managed_user[jose]/File[/home/jose/.bashrc]/content: content changed '{md5}2f8222b4f275c4f18e69c34f66d2631b' to '{md5}19487d0c4a360482b8e74e0480875a85'
      Notice: /Stage[main]/System::Managed_user[jose]/File[/home/jose/.bashrc]/group: group changed 'jose' to 'wheel'
      Notice: Applied catalog in 13.52 seconds
      ```

1. Commit and push your changes.

### Test the effects of your class

1. Switch users on your node to one of the new users: `sudo -su kaitlin`
1. Observe the login script output.

    The output should be similar to this

    ```plaintext
    # sudo -su - kaitlin
    ______________
    < Hello there! >
    --------------
      \   ^__^
       \  (oo)\_______
          (__)\       )\/\
              ||----w |
              ||     ||
    [kaitlin@training root]$ exit
    exit
    #
    ```

## Solution

### Your module structure should resemble

```plaintext
[root@training modules]# tree system/
system/
├── examples
│   └── managed_user.pp
├── files
│   ├── bashrc
└── manifests
    └── managed_user.pp
```

#### Example file: `system/examples/managed_user.pp`

```ruby
# Linux requires a hash for the password. This one is 'Puppet8Labs!'
# Generate your own with the command `openssl passwd -1` if you'd like.
$password = '$1$HdDw//gC$2VBiQ1x5blLPwNS.G.Iw21'

system::managed_user { ['aaron', 'kaitlin', 'alison']:
  password => $password,
}
```

#### Example file: `system/manifests/managed_user.pp`

```ruby
define system::managed_user (
  $password,
) {
  $homedir = $title ? {
    'root'  => '/root',
    default => "/home/${title}",
  }

  # Puppet will evaluate these resources in the proper order because it's smart
  # and knows about dependencies between files and their owners
  user { $title:
    ensure     => present,
    password   => $password,
    managehome => true,
  }

  if $facts['kernel'] == 'Linux' {
    file { "${homedir}/.bashrc":
      ensure => file,
      owner  => $title,
      group  => $title,
      mode   => '0644',
      source => 'puppet:///modules/system/bashrc'
    }
  }
}
```
#### Example file: `system/files/bashrc`

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
|  [Previous Lab](../lab-05.1-Resource-purging)  |  [Next Lab](../lab-06.1-Validating-parameters)  |
