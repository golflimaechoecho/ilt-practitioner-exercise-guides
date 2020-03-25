# Lab 8.1: Create a custom fact

Take a look at your `/etc/krb5.conf` file. This configures the Kerberos network authentication service. It is not configured on these machines, so your default realm should be set to `EXAMPLE.COM`. If we were to write classes that could use Kerberos authentication, it would likely be useful to know what this is set to.

Your task is to write a `default_realm` fact to expose this information as a  global variable.  Using the command line to parse this value might look something like

  ```awk '/default_realm/{print $NF}' /etc/krb5.conf```

## Steps

### Develop your fact

1. Change directory to your `[modulepath]`

    ```$ cd $(puppet agent --configprint environmentpath)/production/modules```

1. Create a new module `kerberos` with the `pdk` command.

    ```$ pdk new module```

1. You will see several questions requiring an answer. Enter the answers as you see below:

    **_Replace the N in studentN with your student number, e.g. `student8`_**

    | Question           | Answer              |
    | ------------------ |:-------------------:|
    | Module Name        | `kerberos`          |
    | Forge Name         | `studentN`          |
    | Credit author      | `Student N`         |
    | License            | `Apache-2.0`        |
    | Operating systems  | RedHat              |

1. Change directories to `kerberos`.
1. Create your `default_realm.rb` custom fact.
    * Edit `lib/facter/default_realm.rb`
    * Execute the sample shell command as a `setcode` string.
1. Syntax check and test your new fact locally.
    * `/opt/puppetlabs/puppet/bin/ruby -c lib/facter/default_realm.rb`
    * `RUBYLIB="kerberos/lib" facter default_realm`

### Deploy and validate your new fact

1. Deploy your codebase.
1. Trigger pluginsync as part of a Puppet run:

    ```puppet agent -t```

1. Review the log output. You should see the md5 hash of the file as it syncs.
1. Syntax check the fact file.

    ```ruby -c $(puppet config print plugindest)/facter/default_realm.rb```

1. Run facter to retrieve the value of your fact.

    ```facter -p default_realm```

**_A full fledged development workstation would likely have Puppet available locally, meaning that you could validate code before deploying and syncing._**

## Solution

### Your module structure should resemble

```plaintext
[root@training modules]# tree kerberos/
kerberos/
└── lib
    └── facter
        └── default_realm.rb
```

#### Example file: `kerberos/lib/facter/default_realm.rb`

```ruby
Facter.add("default_realm") do
  setcode "/bin/awk '/default_realm/{print $NF}' /etc/krb5.conf"
end
```

|  [Previous Lab](../lab-07.2-Export-a-resource)  |  [Next Lab](../lab-08.2-Create-a-custom-function)  |
