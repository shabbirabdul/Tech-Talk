#Chef Sever Installation 

## Install Chef server on Amazon EC2 Ubuntu 14.04

### Download Chef Server 
```
wget https://opscode-omnibus-packages.s3.amazonaws.com/ubuntu/12.04/x86_64/chef-server_11.0.10-1.ubuntu.12.04_amd64.deb
```
### Install the Chef server

`sudo dpkg -i chef-server*`

Then, run the following command to start all of the services

`sudo chef-server-ctl reconfigure`

After the above step, we can access the web interface by typing `https://domain` from our browser. 
  
Because the SSL certificate is signed by an authority not recognized by our browser, we may get a warning. 
Click on the "Proceed anyway" button.

Login with the default admin credentials: 
```  
  username: admin 
  password: p@ssw0rd1
```
When we try to login, it may won't allow, and keep giving us the same window. 
To fix this issue, we need to modify `session_store.rb` file under 

`/opt/chef-server/embedded/service/chef-server-webui/config/initializers`

as follow: 
```
ChefServerWebui::Application.config.session_store :cookie_store, :key => '_sandbox_session', :domain => :all(replace this line with the line below) 
ChefServerWebui::Application.config.session_store :cookie_store, :key => '_sandbox_session', :domain => '#public DNS of your server'
```
Note that we changed `:all` to public domain name. Then, we need to restart chef server using the following command

`sudo chef-server-ctl restart`
