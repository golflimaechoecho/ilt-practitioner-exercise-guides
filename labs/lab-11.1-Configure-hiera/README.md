# Lab 11.1: Configure hiera

Hiera is a data separation layer that allows you to write classes that can easily adapt to customizations per node, per operating system, per environment, or any other facts. This allows the class logic to be straightforward and transparent, while still allowing extreme customizability. Even more so, it allows the end user to simply classify each node and specify parameters using Hiera rules, making the classification process less complicated.

Hiera is a tool that runs on the Puppet master, but it is useful to have a representative data configuration on your local dev workstation for testing. In the classroom, we've set up a local Hiera configuration to pull data from `/etc/puppetlabs/code/hieradata/`. In this lab, you configure the Hiera hierarchy and create datasources; observing how data retrieval works.

**_Looking up data with `puppet lookup` passes the local scope (facts for the local node) for us automatically. To look up data for another node, simply specify that node's name with the `--node <node name>` argument._**

**_Customers on the Long Term Support release can use `puppet apply -e '(notice(lookup("keyname")))'` to perform a similar local lookup._**

As the final step to this lab, you will commit your datasources to your repository,  then push them to the Puppet master. Test that the data retrieval functions still  work when running the Puppet agent.

## Steps

### Configure Hiera with a common datasource

1. Edit the hiera configuration file.
    * `cd /etc/puppetlabs/puppet`
    * Edit `hiera.yaml`
    * set the hierarchy to query `trusted.certname`, `environment`, `common`
1. Edit `common.yaml` in the Hiera datadir.
    * `cd /etc/puppetlabs/code`
    * Edit `hieradata/common.yaml`
    * Take note of the value of the `message` key, or edit as desired.

### Retrieve data from Hiera

1. Use the `puppet lookup` command to display the value of that key
    * `puppet lookup message`

    You can get your hostname by typing `hostname`. This will be used in each step that refers to "yourhost".
    Example, 1954nix0.classroom.puppet.com

### Configure more levels in the hierarchy

1. Create a `production.yaml` in the Hiera datadir.
    * Edit `hieradata/production.yaml`
    * Add a `message` key and set its value to `value from production`
    * Test the hierarchy override by executing the same Puppet code as before.
1. Create <span class="em">yourhost</span> in the hiera datadir.
    * Edit `hieradata/`<span class="em">yourhost</span>
    * Add a `message` key and set the value to <span class="em">yourhost</span>
    * Test the hierarchy override by running the same lookup command as before.

### Deploy and test your datasources

1. Windows Instructions: migrate your datasources to your control repository
    * Copy & paste each file into the corresponding file in your control repository.
    * Edit `[control-repo]/hieradata/common.yaml`
    * Edit `[control-repo]/hieradata/`<span class="em">yourhost</span>
1. Add a `notify` resource to your `site.pp` to display the message.
    * Edit `~/control-repo/manifests/site.pp`
1. Deploy your code.
1. Validate the output of your message.
    * `puppet agent -t`
    * You should observe the same message output.

## Solution

### Example file `~/control-repo/hiera.yaml`

```yaml
---
version: 5
hierarchy:
  - name: Yaml data
    datadir: hieradata
    data_hash: yaml_data
    paths:
      - "%{::trusted.certname}.yaml"
      - "%{environment}.yaml"
      - "common.yaml"
```

**Note:** an equivalent Hiera3 file for LTS versions would be:

```yaml
---
:backends:
    - yaml

:yaml:
    :datadir: /etc/puppetlabs/code/hieradata

:hierarchy:
    - "%{::trusted.certname}"
    - "%{environment}"
    - common
```

### Example file `~/control-repo/hieradata/common.yaml`

```yaml
---
message: 'value from common'
```

### Example file: `~/control-repo/hieradata/production.yaml`

```yaml
---
message: 'value from production'
```

### Example file: `~/control-repo/hieradata/`<span class="em">yourhost</span>

```yaml
---
message: 'value from <span class="em">yourhost</span>'
```

### Example file: `[environment]/manifests/site.pp`

```ruby
node default {
  # ...
  $message = lookup('message')
  notify { $message: }
}
```

|  [Previous Lab](../lab-10.1-Inherited-classes)  |  [Next Lab](../lab-13.1-Designing-profiles)  |
