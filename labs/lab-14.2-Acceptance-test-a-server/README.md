# Lab 14.2: Acceptance test a server

While unit tests validate that class in your module works as expected, we also need a way to ensure that applying these classes together will actually configure a node in the way you expect. Validating that the catalog compiles and includes all the resources expected is of limited usefulness if a managed configuration file has invalid syntax or if the package name passed to the package manager is incorrect. Often, the only way to catch this kind of issue is to actually apply your code on a system and validate the result. We call this an *acceptance test*.

Rspec-puppet is a great tool for unit testing, but it only inspects the contents of a compiled catalog, it has no way of checking the result of applying that catalog to an actual system.

For this kind of testing, we use a tool called serverspec. Serverspec is tool for validating the actual state of a system. It can make sure that a certain service is running, that named users exist, that the machine is listening on the proper ports and responds with the correct response headers, and many other things. Serverspec is only concerned with this end result. In other words, it doesn't validate the *configuration* of a machine, but the *behaviour* of that machine.

In this lab you will create serverspec tests for the Apache module from the previous lab—the one a hapless intern left in your hands. You'll install the required tooling and use the `serverspec-init` tool to generate a skeleton testing framework to start from. Then you'll customize the sample test to fit our needs and validate the end configuration applied by this module. As you work, you may have to resolve issues with both the tests and the code itself.

**_Make sure you've completed at least the setup steps from the unit testing lab before proceeding._**

**_We'll continue using our development environment we set up in the last lab._**

## Steps

**_Best practices are to not pollute Puppet's vendored Ruby path with extra gems unless they're required to directly operate with the Puppet libraries. In this case, Serverspec does not need Puppet libraries, so we'll install to system Ruby._**

### Configure your module for testing
1. Add the Serverspec gem to the Gemfile as the documentation at https://github.com/puppetlabs/pdk-templates#setting-custom-gems-in-the-gemfile
   * update ~/development/apache/.sync.yml

   ```plaintext
   Gemfile:
     optional:
       ':development':
         - gem: 'serverspec'
           version: '>= 2.36.0'
           source: 'https://rubygems.org/'
   ```

   * execute `pdk update --force`

   ```plaintext
   [root@training serverspec]# pdk update --force
   pdk (INFO): Updating studentN-apache using the template at https://github.com/puppetlabs/pdk-templates...

   ----------Files to be modified----------
   Gemfile

   ----------------------------------------

   You can find a report of differences in update_report.txt.


   ------------Update completed------------

   1 files modified.
   ```

1. Create testing directories.
    * `cd ~/development/apache`
    * `mkdir serverspec`
    * `cd serverspec`
1. Create default test files.
    * `pdk bundle exec serverspec-init`
      * OS type: `1) UN*X`
      * Backend type: `2) Exec (local)`

### Run tests and iterate over improvements

1. Ensure that Apache is uninstalled so you can observe a failure.
    * `sudo yum erase httpd`
1. Run the spec tests.
    * `pdk bundle exec rake spec`
1. Enforce the class and run the tests again **_(note the `puppet apply` will fail if you have not completed the fixes from [previous Lab 14.1 Unit test a class](../lab-14.1-Unit-test-a-class))_**.
    * `sudo puppet apply ../examples/init.pp --modulepath ~/development`
    * `pdk bundle exec rake spec`
1. Update your class and/or tests until tests pass.

#### Expected Output

```plaintext
[root@training serverspec]# pdk bundle exec rake spec
pdk (INFO): Using Ruby 2.5.7
pdk (INFO): Using Puppet 6.10.1
/opt/puppetlabs/pdk/private/ruby/2.5.7/bin/ruby -I/opt/puppetlabs/pdk/share/cache/ruby/2.5.0/gems/rspec-core-3.9.0/lib:/opt/puppetlabs/pdk/share/cache/ruby/2.5.0/gems/rspec-support-3.9.0/lib /opt/puppetlabs/pdk/share/cache/ruby/2.5.0/gems/rspec-core-3.9.0/exe/rspec --pattern spec/\{aliases,classes,defines,functions,hosts,integration,plans,tasks,type_aliases,types,unit\}/\*\*/\*_spec.rb
Run options: exclude {:bolt=>true}
......

Finished in 0.07377 seconds
6 examples, 0 failures
```

## Solution

### Your module structure should resemble

```plaintext
[root@training development]# tree -a apache
apache/
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
├── serverspec
│   ├── Rakefile
│   └── spec
│       ├── localhost
│       │   └── sample_spec.rb
│       └── spec_helper.rb
└── spec
    ├── classes
    │   └── apache_spec.rb
    └── spec_helper.rb
```

#### Example file: `apache/serverspec/spec/localhost/sample_spec.rb`

```ruby
require 'spec_helper'

describe package('httpd') do
  it { should be_installed }
end

describe service('httpd') do
  it { should be_enabled }
  it { should be_running }
end

describe port(80) do
  it { should be_listening }
end

# update the conf file in the module or update this
# test with the proper name before this will pass
describe file('/etc/httpd/conf/httpd.conf') do
  it { should be_file }
  it { should contain "ServerName localhost" }
end
```

|  [Previous Lab](../lab-14.1-Unit-test-a-class)  |  [Next Lab](../lab-16.1-Inventory-reports)  |
