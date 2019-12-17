# Lab 16.2 Control puppet with puppet tasks

Puppet's standard model is for the agent to request and apply a new catalog every thirty minutes, by default. This consistency model is sufficient for maintaining system state over time,  but there are cases in which the ability to control nodes in real time or to orchestrate actions across the infrastructure is useful.

We'll experiment with Puppet Tasks to interact with nodes in the classroom and to invoke actions across the classroom infrastructure.

**_Remember that for security reasons, console users must have the "Run Tasks" permission in RBAC in order to run tasks from the console._**

**_This exercise will be performed in the PE Console._**

## Steps:

1. Log in to the Puppet Enterprise console using your user credentials
1. Under "RUN" in the left-hand navigation pane, click "Puppet"
1. In the "Job description" box, enter "Test Puppet run"
1. Under "Run Mode", select "noop"
1. In the "Inventory" dropdown, select "Node list"
1. In the "Add nodes" box, enter your node's hostname.
1. Click "Search"
1. Select your node from the list.
1. A blue footer will appear. Click the "Run job" button in the footer.
1. The run may take a couple of minutes. When it completes, click the link under "Report" to see the results of the Puppet run.
1. When the run completes, click the "Run -> Puppet" link on the left-hand sidebar to return to the "Run Puppet" screen.
1. In the "Job description" box, enter "Test Puppet run"
1. Under "Run Mode", select "noop"
1. In the "Inventory" dropdown, select "Node group"
1. In the "Choose a node group" box, enter "PE Agent (production)".
1. Click "Select"
1. _DO NOT_ run the job, just note the nodes that display in the node list.

#### Discussion Questions
* Did you see any difference in nodes responding when you applied the filter?
* What kind of security considerations might you want to keep in mind when
  architecting your infrastructure?
* What sort of facts might you use rather than encoding roles into a hostname?
  Would you trust facts or hostnames more?
i* When might you use Puppet Tasks to install or update packages instead of Puppet?
  How might you use them together?
  `PE Agent (Production)` class? Why or why not?

|  [Previous Lab](../lab-16.1-Inventory-reports)  |
