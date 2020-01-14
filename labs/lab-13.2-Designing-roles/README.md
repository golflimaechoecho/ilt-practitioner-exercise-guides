# Lab 13.2: Designing roles

Defining technology stacks is a great first start. The configuration of each node can now be described as a list of stacks or profiles that should be included. For example, we might say that each of our *blogserver* nodes is composed of a WordPress profile, a system monitor profile, a log monitoring profile, and--like all of our nodes--a base security profile.

That's much cleaner than describing the full configuration each time, but surely we can do better. This is still tedious and error prone. It's far too easy to inadvertently forget to include one of many profiles when building a new node and that is terrible for consistency and repeatability.

Instead, a better practice is to define *roles* and simply classify each node with the singular role that it serves. For example, we could define the *blogserver* role using the code below and have a consistent and repeatable model for all of our blogserver nodes that is abstracted away from the actual technology stacks used to build it. At any time we could refactor the role or the profiles independently from any other layer in our infrastructure.

```ruby
class role::blogserver {
  include profile::base::security
  include profile::wordpress
  include profile::nagios
  include profile::splunk
}
```

Continue on from this example and add roles for

* An email server node
* A centralized logging server.

Notice that this example includes a profile that wasn't defined in the previous lab. This is to show the iterative nature of designing the different layers. As we identify missing profiles, we can go back and add them to our model, and similarly for component modules included in our profiles.

## Steps

1. If it doesn't exist, create a new `role` directory.
    * `cd [control-repo]/site/`
    * `mkdir -p role/manifests`
1. Create a class in pseudocode or abbreviated code for each role required.
    * Edit `[modulepath]/role/manifests/logserver.pp`
    * Edit `[modulepath]/role/manifests/mailserver.pp`
    * etc.
1. Do not validate and test your classes as they're not intended to be complete
   working solutions.
1. Commit and push your updates.

### Discussion Questions

* What might you do if a few parameters need to be customized in a role or profile?
* What is the benefit to this layering and abstraction?
* What other classification schemes might you see being useful?

## Solution

### Your module structure should resemble

```plaintext
[root@training modules]# tree role/
role/
└── manifests
    ├── blogserver.pp
    ├── logserver.pp
    └── mailserver.pp
```

#### Example file: `role/manifests/blogserver.pp`

```ruby
class role::blogserver {
  include profile::base::security
  include profile::wordpress
  include profile::nagios
  include profile::splunk
}
```

#### Example file: `role/manifests/logserver.pp`

```ruby
class role::logserver {
  include profile::base::security
  include profile::splunk::server
  include profile::nagios
  include profile::splunk
}
```

#### Example file: `role/manifests/mailserver.pp`

```ruby
class role::mailserver {
  include profile::base::security
  include profile::exim
  include profile::nagios
  include profile::splunk
}
```

|  [Previous Lab](../lab-13.1-Designing-profiles)  |  [Next Lab](../lab-14.1-Unit-test-a-class)  |
