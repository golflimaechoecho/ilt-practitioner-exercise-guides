# Lab 7.2: Export a resource

Sharing information across nodes is drastically simplified by exported resources. In this lab, you will accomplish two tasks. First, you will export a host resource for your virtual machine. Second, you will collect all the host resources from the classroom Puppet master and realize them on your node.

As you and other students finish and apply this class, host records for each student's machine will show up in your `/etc/hosts` files. After we have all completed the lab, each machine will be addressable by name from any other machine.

**_This lab relies on resources declared by everyone in the classroom, so if you run into unexplained malformed host records or duplicate resource errors the problem may not be your own code. Please talk among yourselves and request assistance from the instructor as needed._**

## Steps

### Design the classroom host record sharing

1. Add resource exporting and collecting to a `system::classroom` class.
    * Change directory to your `[modulepath]/system`
    * `pdk new class classroom`
    * Edit `manifests/classroom.pp`
    * Declare an exported host record using *facts* for your own node.
    * Tag the resource with `classroom`.
    * Add a resource collector to realize `host` resources tagged with `classroom`.
1. Update your previous class to avoid resource duplication.
    * Edit `manifests/hosts.pp`
    * Comment out the hardcoded record for your own host.
1. Deploy your code.

### Validate the operation of your new class

1. Classify your node group with `system::classroom` in the PE Console.
1. Trigger a puppet agent run with `puppet agent -t`
1. Observe changes in your `hosts` file.
    * `cat /etc/hosts`
    * How many entries are there?
    * What happens if you wait a minute or two and run the agent again?

    **_If you get a `Duplicate declaration` error, you'll need to comment out the `host` resource for your own vm from the `system::hosts` class._**

    The output should be similar to this:

    ```plaintext
    [root@training hosts]# puppet agent -t
    Info: Retrieving plugin
    [...]
    Info: Caching catalog for training.puppetlabs.vm
    Info: Applying configuration version '1379359877'
    Notice: /Stage[main]/Hosts/Host[chris.puppetlabs.vm]/ensure: created
    Info: FileBucket adding {md5}a412e96ac36db22e2fe988313261c71b
    Notice: /Stage[main]/Hosts/Host[fazle.puppetlabs.vm]/ensure: created
    Notice: /Stage[main]/Hosts/Host[clarence.puppetlabs.vm]/ensure: created
    Notice: /Host[clarence]/ensure: removed
    Notice: Finished catalog run in 1.22 seconds
    ```

## Solution

### Your module structure should resemble

```plaintext
[root@training modules]# tree system/
system/
├── examples
│   └── classroom.pp
└── manifests
    └── classroom.pp
```

#### Example file: `system/manifests/classroom.pp`

```ruby
class system::classroom {
  # export a virtual host resource for yourself
  @@host { $::fqdn:
    ensure       => 'present',
    host_aliases => [$::hostname],
    ip           => $::ipaddress,
    tag          => 'classroom',
  }

  # collect all resources from the database (including your own)
  # enforce only those tagged with `classroom`.
  Host <<| tag == 'classroom' |>>
}
```

|  [Previous Lab](../lab-07.1-Ordering-methods)  |  [Next Lab](../lab-08.1-Create-a-custom-fact)  |
