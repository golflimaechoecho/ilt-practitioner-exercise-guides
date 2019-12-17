# Lab 4.1: Validating classification

Puppet's idempotent model makes it easy to write code that maintains configuration indefinitely. Unfortunately, it also makes it more difficult to troubleshoot when changes you expect don't happen when you expect them to. It would be nice to know for sure if Puppet was enforcing the classes you expect, which would help you isolate logic errors.

In this lab, we'll look at some debugging steps that can help you validate whether the code you've written is being enforced properly. We've installed the `jq` tool for parsing the catalog.

**_If you did not complete the `userprefs` section of the Classification exercise earlier, do that first. Alternatively, choose a different class and resources to search for in this exercise._**

## Steps

1. Check the timestamp of your `classfile` with `ls -l $(puppet agent --configprint classfile)`
1. Run Puppet and check the timestamp again to see it change.
1. Ensure that the `userprefs` class was enforced on your node with `grep userprefs $(puppet agent --configprint classfile)`
1. Check to see that the rc file for your shell is being managed:
    * Bash: `grep '/root/.bashrc' $(puppet agent --configprint resourcefile)`
    * Zsh: `grep '/root/.zshrc' $(puppet agent --configprint resourcefile)`

### Inspect the `userprefs` classification parameters

1. Check the catalog to ensure that the parameters you specified are set.

  ```
  $ cd $(puppet agent --configprint client_datadir)/catalog
  $ jq '.resources[] | select(.type == "Class" and .title == "Userprefs") .parameters' $(hostname).json
  ```
  Your output should look similar to this...

  ```json
  {
    "editor": "vim",
    "shell": "bash",
    "gitprompt": true
  }
  ```

1. Inspect the complete `Class` object to see what other interesting data you could use. Press the `up` arrow and edit the command line to remove `.parameters`.

  Your output should look similar to this...

  ```json
  {
    "type": "Class",
    "title": "Userprefs",
    "tags": [
      "class",
      "userprefs",
      "node",
      "master.puppetlabs.vm"
    ],
    "exported": false,
    "parameters": {
      "editor": "vim",
      "shell": "bash",
      "gitprompt": true
    }
  }
  ```

|  [Previous Lab](../lab-03.3-Manage-a-file)  |  [Next Lab](../lab-04.2-Puppet-run-reports)  |