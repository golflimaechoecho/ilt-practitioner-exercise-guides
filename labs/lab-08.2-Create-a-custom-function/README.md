# Lab 8.2: Create a custom function

Writing custom functions allows us to quickly and easily extend the Puppet language to meet our needs. Functions are implemented in Ruby, which is beyond the scope of this class, but this exercise should give you a taste of the power you have at your fingertips. Your task is to write a custom function that accepts a username as a string and returns the expected home directory.

If we were writing code that would run on the node in which the user exists, this would be easy to do with something like:

```plaintext
[root@training ~]# getent passwd "training" | cut -d: -f6
/home/training
```

However, since all functions run on the master, we'll have to make some assumptions about the home directory layout.

The Ruby conditional for this logic might look like

```ruby
case user
when 'root'
  return '/root'
else
  return "/home/#{user}"
end
```

## Steps

1. Create a new function in a ruby file called `homedir.rb`.
    * Edit `[modulepath]/system/lib/puppet/functions/homedir.rb`
1. Validate your code.
    * `ruby -c [modulepath]/system/lib/puppet/functions/homedir.rb`
1. Test your new function using `notify` resources to display the function output.
    * Edit `[control-repo]/manifests/site.pp`
1. Deploy your code.
1. Run a `puppet agent -t`

### Sample test manifest

```ruby
notify { "Root's home directory is ${homedir('root')}": }
notify { "Test's home directory is ${homedir('test')}": }
```

## Extra Credit

Revisit your `system::managed_user` defined type and use this function to calculate the user's home directory.

## Solution

### Your module structure should resemble

```plaintext
[root@training modules]# tree system/
system/
├── examples
│   └── homedir.pp
└── lib
    └── puppet
            └── functions
                └── homedir.rb
```

#### Example file: `system/lib/puppet/functions/homedir.rb`

```ruby
Puppet::Functions.create_function(:homedir) do
  dispatch :homedir do
    param 'String', :use
    return_type 'String'
  end
  # @param [String] user The username to return the conventional Linux home directory of
  # @example Returning the root users homedir
  #   homedir('root') => '/root'
  def homedir(user)
    case user
    when 'root'
      '/root'
    else
      "/home/#{user}"
    end
  end
end
```

#### Example file: `system/examples/homedir.pp`

```ruby
$roothome = homedir('root')
$testhome = homedir('test')

notify { "Root's home directory is ${roothome}": }
notify { "Test's home directory is ${testhome}": }
```

|  [Previous Lab](../lab-08.1-Create-a-custom-fact)  |  [Next Lab](../lab-09.1-Managing-file-content)  |
