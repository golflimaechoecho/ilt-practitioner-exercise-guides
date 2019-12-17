#  Lab 2.1 Editing Code

Review the configuration of the classroom control repository used to deploy your version of code to
your own Puppet environment.

You will use your own branch in the control repository in order to have your
own version of the code. Nearly all the code editing you do in the classroom
will happen in this branch. The instructor will demonstrate the codebase
populating on the classroom Puppet master as each student completes this lab.

**_Throughout the course, we'll refer to the top level of this control
repository as `[control-repo]` when describing paths to files you should edit._**

When we cover community modules from the Puppet Forge, we'll show you how to
incorporate external modules without needing to copy them into your own
repository. The `modules` directory is reserved for these modules.

### Select your own branch of the control repository

1. Log into the classroom welcome page and click the link labeled __Gitlab__
    * Login with the username `studentN`
    * Login with the password `puppetlabs`
    * Click `control-repo` displayed as puppet / control-repo
1. Click the **Production** drop down menu.
1. Use the **Branch** dropdown menu to select your branch.
1. Complete the rest of the lab instructions.


### Clone the control repository

1. The `git config --global user.email` and `git config --global user.name` only need to be run once, no matter how many repositories you clone.
1. The other steps are typical when cloning a repository and only need to be done once per repository.

   **_Run these commands in a Linux shell (or a Windows host machine using Visual Studio code):_**

   ```
   git config --global user.email "noreply@puppet.com"
   git config --global user.name "studentN"
   git clone git@gitlab.classroom.puppet.com:puppet/control-repo.git
   cd control-repo
   git checkout studentN
   ```

1. Edit the Puppetfile inside the control-repo and add the following content.
   **_Replace studentN with your assigned student number (i.e. studentN becomes student0)_**

  ```ruby
  #mod 'pltraining-userprefs'
  #mod 'stahnma/epel'
  #mod 'puppetlabs/apache'
  #mod 'puppetlabs/haproxy'
  #mod 'puppetlabs/mysql'
  #mod 'hunner/wordpress'

  #mod 'review',
  #  :git    => 'git@gitlab.classroom.puppet.com:puppet/review.git',
  #  :branch => 'studentN'

  #mod 'file_content',
  #  :git => 'git@gitlab.classroom.puppet.com:puppet/file_content.git',
  #  :branch => 'studentN'

  #mod 'kerberos',
  #  :git => 'git@gitlab.classroom.puppet.com:puppet/kerberos.git',
  #  :branch => 'studentN'

  #mod 'ordering',
  #  :git => 'git@gitlab.classroom.puppet.com:puppet/ordering.git',
  #  :branch => 'studentN'

  #mod 'system',
  #  :git => 'git@gitlab.classroom.puppet.com:puppet/system.git',
  #  :branch => 'studentN'

  #mod 'time',
  #  :git => 'git@gitlab.classroom.puppet.com:puppet/time.git',
  #  :branch => 'studentN'

  #mod 'webapp',
  #  :git => 'git@gitlab.classroom.puppet.com:puppet/webapp.git',
  #  :branch => 'studentN'
  ```

1. Save it and exit your editor.
1. Commit and push your code changes:

   ```
   git add Puppetfile
   git commit -m 'initial commit'
   git push origin studentN
   ```

|  [Previous Lab](../lab-01.1-Login-welcome-page)  |  [Next Lab](../lab-03.1-Explore-classification)  |