# Lab 9.1: Managing file content

Puppet `file` resources are useful when you need to manage a complete configuration file. Often, however, the resource we want to manage is actually represented by only a portion of a file. One such example is the `host` type. On most Linux systems, host entries are represented by a single line in the `/etc/hosts` file. Puppet doesn't know about all things that can be represented this way. One example is cron's `allow` and `deny` files, which describe which users can run jobs by listing their usernames in a file.

Your first task is to use the `file_line` resource type to ensure that `root` can run cron jobs. You will do this by ensuring that `/etc/cron.allow` has an entry for `root` and that `/etc/cron.deny` has a wildcard (`*`) to deny everyone not explicitly allowed. By managing parts of the file rather than the whole file, you leave the ability to allow access for other users intact.

Your second task is to iteratively build an `/etc/motd` file out of `concat` file fragments. Make sure to include a welcome message header.

Finally, if you choose to tackle the extra credit task, you will abstract the use of the `concat::fragment` resource type by defining your own type that simply accepts a message string and an order number.

**_To prevent resource contention, you will need to remove the `review` classification from your node group if you have not already done so._**

## Steps

1. Change directories to `[modulepath]`

    ```cd $(puppet agent --configprint environmentpath)/production/modules```

2. Create a new module `file_content`

    ```pdk new module```

3. You will see several questions requiring an answer. Enter the answers as you see below:

    **_Replace the N in studentN with your student number, e.g. `student8`_**

    | Question           | Answer              |
    | ------------------ |:-------------------:|
    | Module Name        | `file_content`      |
    | Forge Name         | `studentN`          |
    | Credit author      | `Student N`         |
    | License            | `Apache-2.0`        |
    | Operating systems  | RedHat              |

4. Create the `file_content` class.

    ```plaintext
    cd file_content
    pdk new class file_content
    ```

5. Edit `manifests/init.pp`
6. Add `file_line` resources to manage `cron.allow` and `cron.deny`.
7. Use a `concat` resource to manage `/etc/motd`.
8. Use `concat::fragment` resources to append messages to the `motd`.
    * Use templates to generate content if you'd like.

### Test your module

1. Validate and test your new class

    ```pdk validate```

1. Commit and deploy your codebase.
1. Classify your node with the `file_content` class.
1. Run `puppet agent -t`
1. Validate the results by inspecting `/etc/motd`
1. Run Puppet again and ensure no changes take place.

**_If you forgot to remove the `review` class, you might have noticed surprising behaviour or unexpected content in the `/etc/motd` file. This is because we are trying to manage the same file with two different resource types._**

There are two common ways in which resource contention occurs. The more common is what we call *flapping*, because the same bits on disk are being managed by two different resource types.  Puppet doesn't know about the overlap and attempts to make the file match the state definition in one of the declarations, and then make the file match the state definition of the other declaration. This results in the file content flapping back and forth between two states on every Puppet run.

The resource contention you're likely seeing here is due a quirk in the `concat` resource type that results in the concat resource simply overwriting the content of the file resource.

## Extra Credit

Create a defined type that will allow other classes to register motd messages without needing to use the `concat::fragment` type. An example of using this type might look like:

```ruby
file_content::motd { 'production warning':
    order   => 5,
    message => 'This is a production machine. Please make changes in Puppet instead.',
}
```

## Solutions

### Your module structure should resemble

```plaintext
[root@training modules]# tree file_content/
file_content/
├── examples
│   ├── init.pp
├── manifests
│   ├── init.pp
│   └── motd.pp (optional)
├── templates
    └── motd_header.epp
```

#### Example file: `file_content/manifests/init.pp`

```ruby
class file_content {
  file { '/etc/cron.allow':
    ensure => file,
  }
  file_line { 'prevent cron jobs':
    ensure => present,
    path   => '/etc/cron.deny',
    line   => '*',
  }
  file_line { 'allow root cron jobs':
    ensure => present,
    path   => '/etc/cron.allow',
    line   => 'root',
  }
  concat { '/etc/motd':
    owner => 'root',
    group => 'root',
    mode  => '0644',
  }
  concat::fragment { 'motd header':
    target  => '/etc/motd',
    order   => '01',
    content => epp('file_content/motd_header.epp'),
  }
  concat::fragment { 'sample motd message':
    target  => '/etc/motd',
    order   => '50',
    content => "This is a sample motd message\n",
  }
}
```

#### Example file: `file_content/examples/init.pp`

```ruby
include file_content

# extra credit declaration
file_content::motd { 'maintainer notice':
  message => "\n\n*** This machine is maintained by Wilson ***\n\n",
  order   => 5,
}
```

#### Example file: `file_content/templates/motd_header.epp`

```ruby
Welcome to <%= $fqdn %>

This system is managed by Puppet. Any manual modifications will likely be
overwritten. Make configuration changes in the upstream repository.
```

### Extra Credit Solution

#### Example file: `file_content/manifests/motd.pp`

```ruby
define file_content::motd (
  $message = $name,
  $order   = 10,
) {
  concat::fragment { "motd fragment: ${name}":
    target  => '/etc/motd',
    order   => $order,
    content => $message,
  }
}
```

|  [Previous Lab](../lab-08.2-Create-a-custom-function)  |  [Next Lab](../lab-09.2-Using-augeas)  |
