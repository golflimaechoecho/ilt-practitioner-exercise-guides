# Lab 3.3 Manage a file

Now that we've had some exposure to the `.epp` template format, let's finish the `review::motd` class. If you've been using `.epp` for a while, this will be familiar. If not, you might struggle a bit. Feel free to ask any questions needed or run through the docs page for a brush-up.

https://docs.puppet.com/puppet/latest/lang_template_epp.html

The exercise asks you to add some dynamic content to `/etc/motd` with an `.epp` template. For simplicity, we'll just use a few facts. If you'd rather use your own variables, remember that with `.epp`, you must pass them in with the variables hash.

## Steps

### Update the class from last exercise

1. Edit the file `[modulepath]/review/manifests/motd.pp`.
1. Add a file resource to manage `/etc/motd` using a template to generate the content.
1. Edit the `[modulepath]/review/templates/motd.epp` template in the `review` module.
   * Add a brief welcome message, using the `fqdn` of the host and any other facts you'd like.
1. Deploy your codebase:
   1. `git init`
   1. `git remote add origin git@gitlab.classroom.puppet.com:puppet/review.git`
   1. `git checkout -b studentN`
   1. `git add .`
   1. `git commit -m 'initial commit'`
   1. `git push origin studentN`

### Deploy and Classify

Go to your [control-repo] and edit the Puppetfile by uncommenting the following lines in `[control-repo]/Puppetfile`. When you're done editing, push it to your branch.

```shell
mod 'review',
  :git    => 'git@gitlab.classroom.puppet.com:puppet/review.git',
  :branch => 'studentN'
```

**_You could also use `:branch => :control_branch` which is automatically populated with the control repos branch name (which matches your module repo branch name in this example)._**

#### Commit and push your [control-repo] Puppetfile changes

1. `git add --all`
1. `git commit -m "Add Review Module"`
1. `git push origin studentN`
1. Add classification
    1. Navigate to the **CONFIGURE** -> **Classification** tab.
    1. Find and select your Node Group from the list.
        * `studentN-env environment group`
    1. Add the `review` class and enter a username.
    1. Log into your agent node and enforce the configuration.
        * `puppet agent -t`

**_If your new classes don't show in the PE Console as expected, try validating their syntax using `pdk validate`._**

# Solution

### Your module structure should resemble

```shell
[root@training modules]# tree review/
review/
├── examples
│   ├── motd.pp
│   └── init.pp
├── files
│   └── bashrc
└── manifests
│   ├── motd.pp
│   └── init.pp
└── templates
    └── motd.epp
```

#### Example file: `review/manifests/motd.pp`

```ruby
# A description of what this class does
#
# @summary A short summary of the purpose of this class
#
# @example
#   include review::motd
class review::motd {
  file { '/etc/motd':
    ensure  => file,
    owner   => 'root',
    group   => 'root',
    mode    => '0644',
    content => epp('review/motd.epp'),
  }
}
```

#### Example file: `review/templates/motd.epp`

```ruby
Welcome to <%= $fqdn %>.

This machine is managed with Puppet.
```

|  [Previous Lab](../lab-03.2-Manage-a-file)  |  [Next Lab](../lab-04.1-Validating-classification)  |