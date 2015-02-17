# Tech-Talk
Install Chef server on Amazon EC2 Ubuntu 14.04

Download Chef Server :
wget https://opscode-omnibus-packages.s3.amazonaws.com/ubuntu/12.04/x86_64/chef-server_11.0.10-1.ubuntu.12.04_amd64.deb

Install the Chef server:
sudo dpkg -i chef-server*

Then, we should run the following command to start all of the services:
sudo chef-server-ctl reconfigure

After the above step, we can access the web interface by typing https://domain from our browser. Because the SSL certificate is signed by an authority not recognized by our browser, we may get a warning. Click on the "Proceed anyway" button.

Login with the default admin credentials:
username: admin
password: p@ssw0rd1

When we try to login, it may won't allow, and keep giving us the same window. To fix this issue, we need to modify session_store.rb file under
/opt/chef-server/embedded/service/chef-server-webui/config/initializers file:

ChefServerWebui::Application.config.session_store :cookie_store, :key => '_sandbox_session', :domain => :all(replace this line with the line below)
ChefServerWebui::Application.config.session_store :cookie_store, :key => '_sandbox_session', :domain => '#public DNS of your server'

Note that we changed :all to public domain name.
Then, we need to restart chef server:
sudo chef-server-ctl restart

Chef workstation setup on EC2 Ubuntu 14.04
omnibus installer

The omnibus installer is used to set up a workstation. The omnibus installer uses a single command to install the chef-client and all of its dependencies, including an embedded version of Ruby, RubyGems, OpenSSL, key-value stores, parsers, libraries, and command line utilities. The omnibus installer puts everything into a unique directory (opt/opscode/) so that the chef-client will not interfere with other applications that may be running on the target machine.

To install the chef-client on a workstation, we must run the omnibus installer. To run the omnibus installer, w need to download and run the client installation script from the Chef website:

curl -L https://www.opscode.com/chef/install.sh | sudo bash

When the installation is finished enter the following to verify that the chef-client was installed. When the chef-client is installed correctly, the command shell will return a note that says the version of that was installed:

chef-client -v

install git

git must be installed before the chef-repo can be cloned to the workstation from GitHub:
sudo apt-get update
sudo apt-get install git

Set up the chef-repo

Let's clone the chef-repo skeleton directory.

The chef-repo on GitHub must be cloned to every workstation that will interact with a Chef server. To clone the chef-repo, in a command window, open the home directory, and then clone the chef-repo:
$ cd ~
$ git clone git://github.com/opscode/chef-repo.git

reate .chef Directory

The .chef directory is used to store three files:

  knife.rb
  chef-validator.pem
  admin.pem

These files must be present in the .chef directory in order for a workstation to be able to connect to a Chef server.

To create the .chef directory:

$ sudo mkdir -p ~/chef-repo/.chef
Add .chef to the .gitignore file to prevent uploading the contents of the .cheffolder to GitHub:
$ echo '.chef' >> ~/chef-repo/.gitignore

Login to Chef server

Login to the Chef Server with the admin credentials. we can access the web interface by typing https://domain from our browser. Because the SSL certificate is signed by an authority not recognized by our browser, we may get a warning. Click on the "Proceed anyway" button.

Login with the default admin credentials:

username: admin
password: p@ssw0rd1

After login, we get a new window for password confirmation/change:
Click on "Save User":

Copy the private key and save it in admin.pem file in ~/chef-repo/.chef directory.

Click on the "Clients" tab in the top navigation bar:

Click on the "Edit" button associated with the chef-validator client. Regenerate the private key by selecting that check box and clicking "Save Client":
Copy the private key and save it in the chef-validator.pem file in ~/chef-repo/.chef directory.

Config Knife

Knife is a command-line tool that provides an interface between a local chef-repo and the Chef server. Knife helps provisioning resources, manage recipes/cookbooks, nodes etc.

Create a knife.rb file. This configuration file must be created by running the knife configure --initial command on the machine that will be run as a workstation.

The validation_key attribute in the knife.rb file must specify the path to the validation key. The validation_client_name attribute defaults to chef-validator (which is the chef-validator.pem private key created by the open source Chef server on startup). When prompted for the URL for the Chef server, use the FQDN for the Chef server:

$ sudo knife configure --initial
WARNING: No knife configuration file found
Where should I put the config file? [/home/ubuntu/.chef/knife.rb] /home/ubuntu/chef-repo/.chef/knife.rb
Please enter the chef server URL: [https://ip-172-31-48-124.ec2.internal:443] https://ec2-54-172-41-43.compute-1.amazonaws.com
Please enter a name for the new user: [ubuntu]
Please enter the existing admin name: [admin] 
Please enter the location of the existing admin's private key: [/etc/chef-server/admin.pem] /home/ubuntu/chef-repo/.chef/admin.pem
Please enter the validation clientname: [chef-validator] 
Please enter the location of the validation key: [/etc/chef-server/chef-validator.pem] /home/ubuntu/chef-repo/.chef/chef-validator.pem
Please enter the path to a chef repository (or leave blank): /home/ubuntu/chef-repo
Creating initial API user...
Please enter a password for the new user: 
Created user
Configuration file written to /home/ubuntu/chef-repo/.chef/knife.rb
$ knife user list
Verify that the files are in the .chef folder:

Verify the chef-client install

A workstation is installed correctly when it is able to use knife to communicate with the Chef server. To verify that a workstation can connect to the Chef server:

$ knife user list
should list all users 

$ knife client list

knife.rb
Just for reference, here is knife.rb in /home/ubuntu/chef-repo/.chef/ folder:


log_level                :info
log_location             STDOUT
node_name                'bogotobogo'
client_key               '/home/ubuntu/chef-repo/.chef/bogotobogo.pem'
validation_client_name   'chef-validator'
validation_key           '/home/ubuntu/chef-repo/.chef/chef-validator.pem'
chef_server_url          'https://ec2-54-172-41-43.compute-1.amazonaws.com'
syntax_check_cache_path  '/home/ubuntu/chef-repo/.chef/syntax_check_cache'
cookbook_path [ '/home/ubuntu/chef-repo/cookbooks' ]


