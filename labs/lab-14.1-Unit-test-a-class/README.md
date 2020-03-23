# Lab 14.1: Unit test a class

Getting your modules to work in a test environment is only the first step towards a testing framework that will ensure a reliable, flexible, and maintainable Puppet codebase. As your Puppet envrionment grows in size and complexity, you will need a way to validate that your modules work consistently on multiple platforms and under a variety of conditions.

You will also find that as a codebase gets more complex it becomes exponentially more difficult to change one part of the code without affecting other parts.

As you add features and refactor your code, you will need to ensure that each part of your codebase continues to work correctly acrosss the whole range of conditions in which it might be deployed. To do this, you use an approach called *unit testing*.

A unit test is designed to test small parts of a complete configuration and to test them in isolation from one another. That helps us to identify exactly what pieces have broken and under what conditions. Problems in one class are identified before being conflated with all the other classes in a complete catalog, allowing us to look at it with pinpoint precision, rather than attempting to backtrack through a complete debug log.

This lab is designed to represent a work scenario that plays out far too often. Imagine that an intern has delivered a mostly complete Apache module at the conclusion of his or her internship and it's your job to make this module production ready. You will need to fix any code that doesn't work properly and complete any missing tests to ensure that the code will be maintainable going forward.

In both delivery methods for this class, we'll set up a local development & testing environment separate from your `modulepath`. This will more closely resemble the development workflow for building a single module.

**_All work for this lab will be done locally, even for the virtual form of the class. If you need a Vim refresher, please refer to the cheat sheet at the end of the PDF guide._**

**_The URL to download the `testing_apache.tar.gz` package can be found under the *Downloads* menu in the presentation. Simply right-click and copy the URL._**

## Steps

### Set up the development environment

1. Create your development directory:
    * `mkdir ~/development`
    * `cd ~/development`

### Set up the module for testing

1. Download and uncompress the starter module from the instructor.
    * `wget https://classXXXX-slides.classroom.puppet.com/file/_files/share/testing_apache.tar.gz` (Copy the link from your downloads page)
    * `tar -xvzf testing_apache.tar.gz`
    * `cd apache`
1. Update the RSpec configuration files as required. Run `pdk convert --add-tests`, and when it asks for OS support, choose `RedHat` and `Debian`, accept the changes, and review the files that were added or modified.
1. Note that this module has no external dependencies, so the only optional entry in your `.fixtures.yml` file is  the `symlinks:` section which could be the module itself: `"apache": "#{source_dir}"`
   * Note this is not needed as PDK provides a helper for this, but the `.fixture.yml` file must exist.

### Develop unit tests

1. First we need to validate the tests exist with: `pdk test unit --list`. You should see the list of tests you created during `pdk convert --add-tests` step.

      ```plaintext
      $pdk test unit --list
      Unit Test Files:
      ./spec/classes/apache_spec.rb
      ./spec/classes/params_spec.rb
      ```

      You can even run `pdk test unit`. It will fail, but that's OK for now.

      ```plaintext
      $pdk test unit
      pdk (INFO): Using Ruby 2.5.1
      pdk (INFO): Using Puppet 6.0.2
      [✔] Preparing to run the unit tests.
      [✖] Running unit tests.
      Evaluated 14 tests in 16.385885 seconds: 6 failures, 0 pending.
      ```

      **_The presented output is truncated, discuss the errors with the presenter._**

1. First let us kill some noise in the tests by removing some of the nodes under test specified in our metadata.
   * Edit the `metadata.json` and remove the extra operating systems from the `operatingsystem_support` stanza. These control how many operating systems are tested, so removing them will lower the number of tests.

    ```json
    "operatingsystem_support": [
      {
        "operatingsystem": "RedHat",
          "operatingsystemrelease": [
            "7"
          ]
        },
        {
          "operatingsystem": "Debian",
          "operatingsystemrelease": [
            "8"
          ]
        }
      }
    ],
    ```

    Make sure to validate this, since JSON can be tricky. `pdk validate metadata`

    ```plaintext
    pdk validate metadata
    pdk (INFO): Using Ruby 2.5.1
    pdk (INFO): Using Puppet 6.0.2
    [✔] Checking metadata syntax (metadata.json tasks/*.json).
    [✔] Checking module metadata style (metadata.json).
    ```

    Rerun with `pdk test unit`. We are testing a small subject from the default rspec.

    ```plaintext
    pdk (INFO): Using Ruby 2.5.1
    pdk (INFO): Using Puppet 6.0.2
    [✔] Preparing to run the unit tests.
    [✔] Running unit tests.
        Evaluated 4 tests in 12.777402 seconds: 0 failures, 0 pending.
    ```

    ```ruby
    cat spec/classes/apache_spec.rb

    require 'spec_helper'

    describe 'apache' do
      on_supported_os.each do |os, os_facts|
        context "on #{os}" do
        let(:facts) { os_facts }
        it { is_expected.to compile }
        end
      end
    end
    ```

1. So lets expand that a little bit shall we.
1. First, test that the class itself exists, (https://rspec-puppet.com/documentation/classes/)
   * add `it { is_expected.to contain_class('apache') }` within the context statement.
   * run the PDK test on just the apache class: `pdk test unit --tests ./spec/classes/apache_spec.rb`
   * iterate over this process for each resource (<https://rspec-puppet.com/documentation/classes/#test-a-resource)>
   * try adding `it { is_expected.to contain_service('apache').with('ensure' => 'present', 'enable' => true) }` and run `pdk test unit` It fails, with a clear error message. Correct it and add a package.
   * The package name is different for Debian and RedHat. Review <https://github.com/puppetlabs/puppetlabs-apache/blob/master/spec/classes/apache_spec.rb#L4,> which is the RSpec for the Apache module supported by Puppet.

## Solution

### Your module structure should resemble

**_There will be additional files due to those generated by the PDK._**

```plaintext
[root@training development]# tree -a apache
apache
├── examples
|   └── init.pp
├── files
│   ├── debian.conf
│   └── redhat.conf
├── .fixtures.yml
├── manifests
│   ├── init.pp
│   └── params.pp
├── Modulefile
├── Rakefile
├── README
└── spec
    ├── classes
    │   └── apache_spec.rb
    └── spec_helper.rb
```

#### Example file: `apache/.fixtures.yml`

```yaml
fixtures:
  symlinks:
    "apache": "#{source_dir}"
```

#### Example file: `apache/spec/classes/apache_spec.rb`

```ruby
require 'spec_helper'

describe 'apache' do
  on_supported_os.each do |os, os_facts|
    context "on #{os}" do
      let(:facts) { os_facts }
      it { is_expected.to contain_service('apache').with('ensure' => 'running', 'enable' => true) }
      it { is_expected.to contain_class('apache') }
      it { is_expected.to compile }
    end
  end
end
```

#### Example file: `apache/manifests/init.pp`

```ruby
# == Class: apache
#
# Full description of class apache here.
#
# === Parameters
#
# Document parameters here.
#
# [*sample_parameter*]
#   Explanation of what this parameter affects and what it defaults to.
#   e.g. "Specify one or more upstream ntp servers as an array."
#
# === Variables
#
# Here you should define a list of variables that this module would require.
#
# [*sample_variable*]
#   Explanation of how this variable affects the funtion of this class and if it
#   has a default. e.g. "The parameter enc_ntp_servers must be set by the
#   External Node Classifier as a comma separated list of hostnames." (Note,
#   global variables should not be used in preference to class parameters  as of
#   Puppet 2.6.)
#
# === Examples
#
#  class { apache:
#    servers => [ 'pool.ntp.org', 'ntp.local.company.com' ]
#  }
#
# === Authors
#
# Author Name <author@domain.com>
#
# === Copyright
#
# Copyright 2013 Your name here, unless otherwise noted.
#
class apache inherits apache::params {
  package { 'apache':
    name   => $apache::params::package,
    ensure => present,
  }
  file { 'apache_config':
    path    => $apache::params::config,
    ensure  => file,
    # Use $osfamily instead of $operatingsystem and double quote
    source  => "puppet:///modules/apache/${::osfamily}.conf",
    require => Package['apache'],
  }
  service { 'apache':
    name      => $apache::params::service,
    ensure    => running,
    enable    => true,
    subscribe => File['apache_config'],
  }
}
```

|  [Previous Lab](../lab-13.2-Designing-roles)  |  [Next Lab](../lab-14.2-Acceptance-test-a-server)  |
