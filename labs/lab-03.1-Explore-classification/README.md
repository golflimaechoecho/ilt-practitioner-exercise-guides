# Lab 3.1: Explore classification

In this lab you are configuring a node group on the master to map your Puppet agent to this environment with your own code. Running `puppet agent -t` on your container will initiate a Puppet agent run and enforce the configuration specified in your environment. Note that we are adding classification to an *Environment Group* due to this being an academic setting. This is not best practices for a production setting.

Then customize root's user account on your agent by adding classification rules to your node group with parameters of your choosing. Classification options are available from both your personal environment and the global codebase.  You will be choosing the default login shell and default editor.

**_Reminder: Any time you see classXXXX, it is referring to your class id, such as class1809._**

**_Reminder Any time you see studentN, it is referring to your assigned student number, such as student4._**

### Examine your node in the Console

1. Log into the Enterprise Console using your web browser:
    * Navigate to the welcome page and click on the Console URL
    * Log in with your user account credentials.
    * username: studentN
    * password: puppetlabs
1. Sign your agents certificate that was created in Lab 1.1
    * Click on the *Unsigned Certs* tab on the left nav bar.
    * Sign only your certificate by choosing Accept, DO NOT select Accept All.
1. Explore the various data Puppet provides about the classroom infrastructure.
    * Click on the *INSPECT -> Overview* tab to see the latest classroom status.
    * Use the interface to determine whether any Puppet runs have failed.
    * Find your node and look at the containment graph.
1. Examine the details of your own node.
    * Click on the *INSPECT -> Nodes* tab
    * Find your own node in the inventory list.
        * Inspect your facts listing.
        * Determine which groups your node is classified in.
        * Look at historical reports for your node, including the runtime statistic.
    * Compare with the classroom master's details.

### Create an environment group

The first step is to create a new node group that will have the environment group checkbox selected. This group will assign your nodes to run in your environment (studentN), which corresponds to your branch in the control-repo.

* Click on Classification in the left menu.
* Click **Add group...** at the top.
* Enter the following (Where N is your student number):
  * Parent name: `All Environments`
  * Group name: `studentN-env`
  * Environment: `studentN`
  * Environment Group: `checked`
  * Description: `This group sets the environment for studentN`
  * Click **Add**
* Now that the group has been created, expand the `All Environments` group (click the **+** sign to the left of the name) and click on your new **studentN-env** group
* On the `Rules` tab, navigate to the section that says `Pin specific nodes to the group`
* Click on **Node name** and select only your Linux machine from the list
* Finally, click on **Commit X changes** in the lower right hand corner

### Classify your node group with customizations

Go to your [control-repo] and edit the Puppetfile. Uncomment the userprefs module and its dependencies.

```
mod 'pltraining-userprefs'
mod 'stahnma/epel'
```

#### Commit & push your [control-repo] Puppetfile changes

These commands are the same on windows and linux:

1. `git add Puppetfile`
1. `git commit -m 'Add userprefs module'`
1. `git push origin studentN`

#### Classify the userprefs module

1. Return to the PE Console (linked on the welcome page).
1. In your studentN-env environment, click on the **Configuration** tab.
1. Add the `userprefs` class by typing its name into the `Add new class` text box.
1. Add parameters to the class using the dropdown list.
1. Choose an editor by typing into the parameter text box.
    * Choose from `mg` (a tiny Emacs clone), `nano`, or `vim`
1. Choose a login shell by typing into the parameter text box.
    * Choose from `bash` or `zsh`
1. Commit changes by pressing the button.
1. Enforce your configuration on your Linux agent.
    * `puppet agent -t`

|  [Previous Lab](../lab-02.1-Editing-code)  |  [Next Lab](../lab-03.2-Create-a-module)  |
