# Lab 16.1: Inventory reports

You can run tasks from the Puppet Enterprise console in order to perform actions on demand in your infrastructure. The facter_task task gathers inventory information about managed nodes by invoking facter. To get a full list of all facts known about a node, simply invoke facter_task against that node's name. In this exercise, you will use facter_task to gather inventory information about the classroom and display it in the PE console.

**_Remember that for security reasons, console users must have the "Run Tasks" permission in RBAC in order to run tasks from the console._**

## Steps

1. Log in to the Puppet Enterprise console using your user credentials
1. Under "RUN" in the left-hand navigation pane, click "Task"
1. In the "Task" box, enter "facter_task"
1. In the "Inventory" dropdown, select "Node Group"
1. In the "Choose a node group" box, enter "All nodes" (production)
1. Click "Select"
1. A blue footer will appear. Click the "Run job" button in the footer.
1. Return to the Tasks page. This time, select only the Puppet master by running a PQL query as follows:
    * In the "Task" box, enter "facter_task"
    * In the "Inventory" dropdown, select "PQL Query"
    * In the "Common queries" dropdown, select "Nodes with a specific resource"
    * Modify the query text to read `resources[certname] { type = "Service" and title = "pe-puppetserver" }`
    * Click "Submit query"
    * A blue footer will appear. Click the "Run job" button in the footer.

### Expected Output

The output should be similar to this:

```plaintext
master.puppetlabs.vm          Start time: 2018-06-28 00:37 Z Run time: 00:00:36 Succeeded
{
  "staging_http_get" : "curl",
  "has_old_shiro_ini_default" : false,
  "path" : "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin",
  "puppet_vardir" : "/opt/puppetlabs/puppet/cache",
  "identity" : {
    "gid" : 0,
    "uid" : 0,
    "user" : "root",
    "group" : "root",
    "privileged" : true
  },
<snip>
```

|  [Previous Lab](../lab-14.2-Acceptance-test-a-server)  |  [Next Lab](../lab-16.2-Control-puppet-with-puppet-tasks)  |
