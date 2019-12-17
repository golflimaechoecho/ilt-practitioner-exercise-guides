# Lab 9.2: Using augeas

Augeas abstracts the responsibility of text file parsing away from us. Rather than intimately understanding how to parse and modify every configuration file we want to manage, we can leave the file format to Augeas and simply manage settings individually.

In this lab, you will use the `augeas` resource type to ensure that the Kerberos default realm we looked at before is set to `PUPPETLABS.VM`.

## Steps:

### Discover the context for the setting we want to manage

Use the `/opt/puppetlabs/puppet/bin/augtool` shell to find the path to `default_realm`
in `krb5.conf`. Remember that the shell will tab-complete.

```
[root@training ~]# /opt/puppetlabs/puppet/bin/augtool
augtool> ls /files/etc/krb5.conf
logging/ = (none)
libdefaults/ = (none)
realms/ = (none)
domain_realm/ = (none)
augtool> get /files/etc/krb5.conf/libdefaults/default_realm
/files/etc/krb5.conf/libdefaults/default_realm = EXAMPLE.COM
augtool> quit
```

### Write a class to manage Kerberos

You have already created a kerberos class, you will be modifying it now.

1. Add a new `augeas` resource type to your class.

    ```pdk new class augeas```

    * Configure the `context` attribute to the path you discovered for `default_realm`
    * Set the `changes` attribute to `set` the value of `default_realm` to `PUPPETLABS.VM`
1. Validate your new class:

    ```pdk validate```
1. Commit your code
1. In the PE console, classify your node with 'kerberos'.
1. Run `puppet agent -t`
1. Validate your change using the `default_realm` custom fact or by inspection.
    ```
    facter -p default_realm
    grep default_realm /etc/krb5.conf
    ```

    The output should be similar to this:

    **_Depending on the Kerberos version installed, you may notice that your fact now returns multiple settings! Some packages ship with the `DEFAULT_REALM` rule commented out and this confuses our simple text parsing fact from the earlier lab. You may consider implementing the Augeas based fact described in the extra credit options as a more robust solution._**

    ```
    [root@training modules]# puppet apply kerberos/examples/init.pp
    notice: /Stage[main]/Kerberos/Augeas[krb5.conf]/returns: executed successfully
    notice: Finished catalog run in 0.20 seconds
    [root@training modules]# facter -p default_realm
    PUPPETLABS.VM
    ```

## Extra Credit option(s)

#### Use Augeas libraries in a fact

Modify the existing shell based fact from an earlier lab to retrieve the default realm in a less fragile manner than manual text parsing. Example Ruby code might look like the following. Recall that Ruby, like perl, implicitly returns the value of the last expression evaluated.

```ruby
require 'augeas'
Augeas::open do |aug|
  aug.get('/files/etc/krb5.conf/libdefaults/default_realm')
end
```

#### Create a defined type

Create a defined type that will make it easy to modify the `default_realm` or
other Kerberos settings without using Augeas directly.

An example of using this type might look like:

```ruby
kerberos::defaults { 'default_realm':
  value => 'PUPPETLABS.VM',
}
kerberos::defaults { 'ticket_lifetime':
  value => '12h',
}
```

# Solution

### Your module structure should resemble:

```
[root@training modules]# tree kerberos/
kerberos/
├── examples
│   │── defaults.pp
│   └── init.pp
├── lib
│   └── facter
│       └── default_realm.rb
└── manifests
    └── init.pp
```

#### Example file: `kerberos/manifests/init.pp`

```ruby
class kerberos {
  augeas { 'krb5.conf':
    context => '/files/etc/krb5.conf/libdefaults',
    changes => 'set default_realm PUPPETLABS.VM',
  }
}
```

#### Example file: `kerberos/examples/init.pp`

```ruby
include kerberos
```

### Extra Credit Solution

#### Example file: `kerberos/lib/facter/default_realm.rb`

```ruby
require 'augeas'
Facter.add('default_realm') do
  setcode do
    Augeas::open do |aug|
      aug.get('/files/etc/krb5.conf/libdefaults/default_realm')
    end
  end
end
```

|  [Previous Lab](../lab-09.1-Managing-file-content)  |  [Next Lab](../lab-10.1-Inherited-classes)  |