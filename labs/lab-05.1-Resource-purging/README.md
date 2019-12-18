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
* Change the current working directory to the module you created.

  ```cd system```

### Resource purging

1. Gather the current hosts by running `puppet resource host`
1. Turn the output of the current hosts into a Puppet class file.
    * `pdk new class hosts`
    * Edit `manifests/hosts.pp`
    * Paste the output of the `puppet resource host` command into the class.
1. Add a `resources` resource to the class.
    * Set the `purge => true` attribute.
1. Validate and test the new class.

    ```pdk validate```

1. Commit your changes to the repo.
    * Init your module: `git init`
    * Add the origin: `git remote add origin git@gitlab.classroom.puppet.com:puppet/system.git`
    * Checkout the branch: `git checkout -b studentN`
    * Add the files: `git add --all`
    * Create a commit: `git commit -m "Initial Commit"`
    * Push the commit to the Git server: `git push origin studentN`
1. Change directories to `[control-repo]`
    * Edit the `Puppetfile` and uncomment out these lines.  

      ```
      mod 'system',
        :git => 'git@gitlab.classroom.puppet.com:puppet/system.git',
        :branch => 'studentN'
      ```

1. Commit and push your control repo changes, using the git add/commit/push sequence.

### Log in to the Puppet Enterprise console and classify your new class

1. Point your browser at https://classXXXX-master.classroom.puppet.com/

1. Enter the credentials:
    * **username:** *studentN*  
    * **password:** *puppetlabs*

1. In the left menu, under **CONFIGURE**, click **Classification**
1. Open `All Environments` and your `studentN-env`
1. Click on the `Configuration` tab
1. Under `Classes` section, `Add new class` type in `system::hosts` and `Add class`
1. Click **Commit 1 change** in the lower right corner.

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

1. Run puppet to purge your change: `puppet agent -t`
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
