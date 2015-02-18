#Install knife ec2 plugin on workstation

The knife ec2 is used to manage API-driven cloud servers that are hosted by Amazon EC2. Chef Knife plugin for EC2 gives knife the ability to create, bootstrap, and manage EC2 instances.

## Install Knife
To install the knife ec2 plugin using RubyGems, we want to run the following command:
```
$ sudo apt-get install make g++ $ sudo apt-get install libxml2 libxml2-dev libxslt1-dev 
$ sudo apt-get install zlib1g-dev $ sudo /opt/chef/embedded/bin/gem install knife-ec2
```
## Uploading cookbooks

1. On the workstation, create the required cookbooks and upload to the Chef Server. 
  
    `$ cd ~/chef-repo/cookbooks/`

2. Upload the cookbooks to the server:

    `$ knife cookbook upload java apt $ knife cookbook upload chef-sugar`

## Create a Role Using JSON

We can use JSON to configure roles.

A role is a way to define certain patterns and processes that exist across nodes in an organization as belonging to a single job function. Each role consists of zero (or more) attributes and a run-list. Each node can have zero (or more) roles assigned to it. When a role is run against a node, the configuration details of that node are compared against the attributes of the role, and then the contents of that role's run-list are applied to the node's configuration details. When a chef-client runs, it merges its own attributes and run-lists with those contained within each assigned role.

We will create a role named "my_app" using the following command:

    export EDITOR=$(which vi) $ knife role create my_app

Then, at the knife role create command, an editing window will be opened.

In the editor, fill the file contents in JSON format as below: 
```
{ "name": "my_app", "description": "", "json_class": "Chef::Role", "default_attributes": {
}, "override_attributes": {
}, "chef_type": "role", "run_list": ["recipe[apt]", "recipe[tomcat]"], "env_run_lists": {
} }
```

After finishing edit, we get:

    Created role[my_app]

## Bootstrap an EC2 instance

Now we are all set to bootstrap an EC2 instance with the my_app role with the following command:
```
$sudo knife ec2 server create -I ami-9eaa1cf6 -r "role[my_app]" -Z us-east-1c -g Chef11_Server_On_EC2_Security_Group --ssh-key Chef_11_Server -i Chef_11_Server.pem -f t2.micro -A 'AK' -K "bm" --region us-east-1 --ssh-user ubuntu
```
