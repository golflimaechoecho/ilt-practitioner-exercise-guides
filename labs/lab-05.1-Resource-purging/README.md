# Lab 5.1: Resource purging

If we think about the set of a certain type of resources on our system as being part of a list, we can tell Puppet that we want to explicitly manage the whole set. In other words, if Puppet isn't managing a resource in that set then it will be removed. This clearly only works when that set is a list of finite length, such as the list of packages or the list of users.

There is a finite set of host records stored in `/etc/hosts`. Create a Puppet manifest that represents them using the output of a `puppet resource` command. Then use a `resources` resource to purge unmanaged host entries and experiment to see the effects this has.

If you damage the hosts file and cannot reach the master, let the instructor know and they will help you restore it.

**_Tip: You can redirect the output of a command to a file, if it'sconvenient. For example, `puppet resource host > hosts.pp`._**

**_Change the current working directory to the `environmentpath`_** 

  ```cd $(puppet agent --configprint environmentpath)/production/modules```

### Create the `system` module

* Create a new PDK module called `system`

  ```$ pdk new module```

  * You will see several questions requiring an answer. Enter these answers:

  **_Replace the “N” in these answers with your student number (for example, `student8`)._**

  | Question           | Answer              |
  | ------------------ |:-------------------:|
  | Module Name        | `system`            |
  | Forge Name         | `studentN`          |
  | Credit author      | `Student N`         |
  | License            | `Apache-2.0`        |
  | Operating systems  | RedHat              |

* Press `Y` to generate the module in the current directory.
* Change the current working directory to the module you created, then use the `pdk` command to create a new class called `hosts`.

  ```cd system```

* At the command prompt, type

  ```$ pdk new class hosts```

* Edit `modules/hosts.pp` and add the content from the command

  ```puppet resource host```

### Deploy your new module

To allow your module to be shared among your colleagues and Puppet masters, you need to push it to a `remote` git server. The following set of commands adds the URL to push to using the name `origin`.

In the base of your system module, run these commands to create your own branch and push your code:

1. Init your module: `git init`
1. Add the origin: `git remote add origin git@gitlab.classroom.puppet.com:puppet/system.git`
1. Checkout the branch: `git checkout -b studentN`
1. Add the files: `git add --all`
1. Create a commit: `git commit -m "Initial Commit"`
1. Push the commit to the Git server: `git push origin studentN`

### Validate your new module

Verify your system module branch is now in Gitlab.

1. In a browser, navigate to
   `https://classXXXX-gitlab.classroom.puppet.com/puppet/system/tree/studentN`
1. You should see the files you previously committed.

  **_If prompted to sign-in, use the username `studentN` and the password `puppetlabs`._**

### Resource purging

1. Gather the current hosts by running `puppet resource host`
1. Turn the output of the current hosts into a Puppet class file.
    * `pdk new class hosts`
    * Edit `manifests/hosts.pp`
    * Paste the output of the `puppet resource host` command into the class.
1. Add a `resources` resource to the class.
    * Set the `purge => true` attribute.
1. Validate and test the new class.
    * `pdk validate`
    * `git add .`
    * `git commit -m 'update hosts class'`
    * `git push origin studentN`
    * As we did with the review module, uncomment out these lines in the [control-repo]/Puppetfile:
      
      ```
      mod 'system',
        :git => 'git@gitlab.classroom.puppet.com:puppet/system.git',
        :branch => 'studentN'
      ```

    * Commit and push your control repo changes, using the git add/commit/push sequence.
    * Run `puppet agent -t` on your Linux node.
    * No changes should be observed, aside from cosmetic changes such as `'::1'` to `'127.0.0.1'`
1. Ensure you add `system::hosts` in PE classification for your studentN environment.

### Observe purging
1. Add a new entry in `/etc/hosts` named `removeme.puppetlabs.vm`
    * Add `1.2.3.4   removeme.puppetlabs.vm`
    
    **Sample `/etc/hosts` file:**

    ```
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

    10.120.0.235 puppet.classroom.puppet.com puppet
    10.120.0.51 gitlab.classroom.puppet.com gitlab gitlab.puppet.vm
    1.2.3.4 removeme.puppetlabs.vm
    ```

1. Run puppet again to purge your change: `puppet agent -t`
    **Expected output:**

    ```
    [root@1905nix0 control-repo]# puppet agent -t
    ...
    Info: Caching catalog for 1905nix0.classroom.puppet.com
    Info: Applying configuration version 'puppet-student0-5862f9c2c38'
    Notice: /Stage[main]/System::Hosts/Host[removeme.puppetlabs.vm]/ensure: removed
    Info: Computing checksum on file /etc/hosts
    Notice: Applied catalog in 0.29 seconds
    ```

# Solution

### Your module structure should resemble

```shell
[root@training modules]# tree system/
system/
├── examples
│   └── hosts.pp
└── manifests
    └── hosts.pp
```

#### Example file: `system/manifests/hosts.pp`

```ruby
class system::hosts {
  resources {'host':
    purge => true,
  }
  host { 'master.puppetlabs.vm':
    ensure       => present,
    host_aliases => ['master'],
    ip           => '192.168.X.X', ## use the classroom IP
  }
  host { 'localhost':
    ensure       => present,
    host_aliases => [
                      'localhost.localdomain',
                      'localhost4',
                      'localhost4.localdomain4',
                      'training.puppetlabs.vm',
                      'training'
                    ],
    ip           => '127.0.0.1',
  }

## Use your own IP, or the ::ipaddress fact
  host { 'yourname.puppetlabs.vm':
    ensure       => present,
    host_aliases => ['yourname'],
    ip           => $::ipaddress,
  }
}
```

#### Example file: `system/examples/hosts.pp`

```ruby
include system::hosts
```

|  [Previous Lab](../lab-04.2-Puppet-run-reports)  |  [Next Lab](../lab-05.2-Defined-type)  |
