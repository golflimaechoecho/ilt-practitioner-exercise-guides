# Lab 4.2: Puppet run reports

Sometimes you won't have the ability to watch a Puppet run proceed. You will need to read the reports to see what happened. The Puppet Enterprise Console provides an interface for doing so, but sometimes it's useful to write scripts to show changes or to view more data than is visible in the Console. For this, we will look at the raw YAML report on the node itself.

The report configuration version can provide history as to which version of the  Puppet codebase was used to generate the catalog applied. This allows you to correlate code changes to configuration changes.

In this lab, you'll look at your cached report and configure your environment's `config_version`.

## Steps

1. See what changes were made in the last Puppet run
    
    ```grep -B 25 'changed: true' $(puppet agent --configprint lastrunreport)```

    Your output should look similar to this...

    ```shell
    ...
      change_count: 0
      out_of_sync_count: 0
      events: []
      corrective_change: false
    File[/root/.bashrc.puppet]: !ruby/object:Puppet::Resource::Status
      title: "/root/.bashrc.puppet"
      file: "/etc/puppetlabs/code/modules/userprefs/manifests/bash.pp"
      line: 42
      resource: File[/root/.bashrc.puppet]
      resource_type: File
      containment_path:
      - Stage[main]
      - Userprefs::Bash
      - File[/root/.bashrc.puppet]
      evaluation_time: 0.003285356
      tags:
      - file
      - class
      - userprefs::bash
      - userprefs
      - bash
      - node
      - default
      time: '2017-03-17T19:13:20.811488070+00:00'
      failed: false
      changed: true
    ```

1. If no changes appeared, make a minor modification to a managed file and run a `puppet agent -t`.
    * `vim /root/.bashrc.puppet`, or
    * `vim /root/.zshrc.puppet`
    * `puppet agent -t`
1. Take a look at the raw data in a report.
    * `vim $(puppet agent --configprint lastrunreport)`

### Validate your existing environment version

1. Run the agent and check its configuration version: `puppet agent -t`

  Your output should look similar to this...

  ```
  [root@training ~]# puppet agent -t
  Notice: Local environment: 'production' doesn't match server specified node environment 'training', switching agent to 'training'.
  Info: Retrieving pluginfacts
  Info: Retrieving plugin
  Info: Loading facts
  Info: Caching catalog for training.puppetlabs.vm
  Info: Applying configuration version '1489778215'
  ```

### Configure a `config_version` for your environment
1. Validate the `config_version` of your environment by reviewing the `environment.conf` file: `cat ~/control-repo/environment.conf`

  **Example file: `[environment]/environment.conf`**

  ```shell
  modulepath = site:modules:$basemodulepath
  config_version = 'scripts/config_version.sh $environmentpath $environment'
  ```

2. Run the agent and check its new configuration version: `puppet agent -t`
  Your output should look similar to this...

  ```
  [root@training ~]# puppet agent -t
   Notice: Local environment: 'production' doesn't match server specified node environment 'training', switching agent to 'training'.
  Info: Retrieving pluginfacts
  Info: Retrieving plugin
   Info: Loading facts
  Info: Caching catalog for training.puppetlabs.vm
  Info: Applying configuration version '`#{compiling_master}-#{environment}-#{commit_id}`'
  ```
3. Compare the configuration version to that of your development repository.
    * Linux: `git rev-parse --short HEAD`
    * Windows: see the most recent commit listed for your branch of the control repository.

|  [Previous Lab](../lab-04.1-Validating-classification)  |  [Next Lab](../lab-05.1-Resource-purging)  |
