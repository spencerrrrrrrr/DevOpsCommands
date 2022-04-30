# Significant Code Lines
[GitHub Flavored Markdown Cheat Sheet](https://gist.github.com/stevenyap/7038119)
>Format contributions to GitHub Flavored Markdown

## **Apache2**

`sudo systemctl status apache2`
>Check the running status

`sudo systemctl is-enabled apache2`
>Check that Apache2 is enabled

`sudo systemctl disable apache2`
>Disable Apache2

`sudo systemctl stop apache2`
>Stops Apache2 from running

`sudo systemctl mask apache2`
>Links Apache2 unit files to /dev/null, making it impossible to start it

## **AWS CLI/CloudShell**
>If a command does not work, check that you have the appropriate permissions to perform the command and that your AWS CLI is up to date.

`aws iam list-users`
>Lists all the users that have been created in IAM.

## **Ansible**
***Target users must have sudoer privileges in /etc/sudoers/
ansible_sudo_pass="yourPassword" >Place in inventory.txt file
ansible_host=192.168.x.x >Place in inventory.txt file. Assigns IP to target name
ansible_connection=ssh >Establishes connection type, "ssh" is Linux-based***

```
vars:
   variable-name: variable-assignment #Such as an ip address: 192.168.x.x.
```
>Format to create a variable. Variables must be called in double curly braces {{ variable-name}}

```
apt:
  pkg:
    python
    apache
    etc.
```
>Format to install packages in ubuntu via Ansible

## **Chef**
[Resources](https://docs.chef.io/resources/)
>Resources used in Chef are listed under the Infra portion of the Workstation
**Be sure to disable Apache2 or anything else that might be using ports 80 or 443**

**Setting Up Chef Server**
---
```
sudo ufw allow proto tcp from any to any port 80
sudo ufw allow proto tcp from any to any port 443
```
>Open ports for Chef to use

```
sudo vim /etc/hosts
127.0.0.1 chef-server.example.com
```
>If you have an active DNS server, set the A record accordingly. For installations without DNSserver, set the record on /etc/hosts file

```
sudo apt -y install chrony
sudo timedatectl set-timezone America/New_York
sudo systemctl restart chrony
sudo timedatectl set-ntp true
```
>Because Chef is prone to time drift, it's best to connect your system to Network Time Protocol (NTP). `set-ntp true` ensures ntp synchronization

```
VERSION="14.12.21"
wget https://packages.chef.io/files/stable/chef-server/${VERSION}/ubuntu/18.04/chef-server-core_${VERSION}-1_amd64.deb
sudo apt install ./chef-server-core_${VERSION}-1_amd64.deb
```
>Installs Chef Server 2/1/22

```
sudo chef-server-ctl reconfigure
sudo chef-server-ctl user-create USER_NAME FIRST_NAME LAST_NAME EMAIL 'PASSWORD' --filename FILE_NAME
```
>Creates an admin account and auto generates an rsa key. FILE_NAME is where the key will be stored - this is not the same key as the one generated with org-create. Password must be added within quotes.

`sudo chef-server-ctl org-create short_name 'full_organization_name' --association_user user_name --filename /path/to/org-validator.pem`
>Creates an organization account. Short name must be all lowercase letters or digits, and the full name can't start with a whitespace. user_name associates the specified user with the admins security group on the Chef server. --filename specifies where to save the rsa private key that is generated with this command - not the same name as the key generated with user-create.

**Setting Up Chef Manage** *On Chef Server*
---
```
sudo chef-server-ctl install chef-manage
sudo chef-server-ctl reconfigure
```
**or**
```
VER="3.2.20"
wget https://packages.chef.io/files/stable/chef-manage/${VER}/ubuntu/18.04/chef-manage_${VER}-1_amd64.deb
sudo apt install -f ./chef-manage_${VER}-1_amd64.deb
sudo chef-manage-ctl reconfigure
```
>Installs the Chef management console, **WHICH IS ONLY FREE FOR UP TO 25 NODES**. This allows GUI access from the browser.

**Setting Up Chef Workstation**
---
```
sudo ufw allow proto tcp from any to any port 80
sudo ufw allow proto tcp from any to any port 443
```
>Opens ports 80 and 443 because those are the ports Chef is using

```
VER="22.1.745"
wget https://packages.chef.io/files/stable/chef-workstation/${VER}/ubuntu/20.04/chef-workstation_${VER}-1_amd64.deb
sudo dpkg -i chef-workstation_*.deb
rm chef-workstation_*.deb
```
>Installs Chef Workstation for Ubuntu 20.04 2/1/22. If the dpkg command
[Chef Workstation Downloads](https://www.chef.io/downloads/tools/workstation)

```
chef generate repo chef-repo
cd chef-repo
```
>Generates a Chef repo on your Workstation machine

*If you move .chef into the chef-repo, be sure to create a .gitignore file and add .chef to it.*

`scp chef-server-ip:/home/user-or-group-validator.pem .`
>Perform on Workstation. Grabs the public key from the Chef Server.

*Create a knife.rb file*
`knife configure init-config`
>Creates a config file after prompting the user for info to link the workstation to the server.

```
current_dir = File.dirname(__FILE__)
log_level :info
log_location STDOUT
node_name "user-name"
client_key "path/to/key-directory/user-name.pem"
validation_client_name   'organization-name-validator'
validation_key "organization-name-validator.pem"
chef_server_url "https://chef-server.example.com_or-ip-address/organizations/perscholas"
cache_type 'BasicFile'
cache_options(:path => "#{ENV['HOME']}/.chef/checksums")
#My build did not like the "=>" in the cache_options, but it seemed to work for some. Include it, then check compatibility with the command "cookstyle knife.rb". If your system doesn't like it, you can remove that or autocorrect with cookstyle -a knife.rb
cookbook_path ["#{current_dir}/../cookbooks"]
```
>Formatting for knife.rb, also known as config.rb. Either of these file names will be recognized as knife configuration.

`knife ssl fetch`
>Fetch the SSL certificate from your Chef server. Perform this command from the directory your keys are in

`knife ssl check`
>Validates SSL certificates from the Chef server

`file trusted_certs/chef-server.crt`
>Displays more detail about the file type, in this case it would output `trusted_certs/chef-server.crt: PEM certificate`

`knife client list`
>Checks knife's client list

`knife bootstrap 192.168.x.x --ssh-user node-username ssh-password username-password -N node_name --sudo --use-sudo -P sudo-password`
>Bootstraps a node to be recognized by the Chef Server. -N seems to have better results than --node_name, although knife bootstrap --help suggests they are the same.

`knife node list`
>Lists all the nodes recognized by the Chef server

`knife node show node-name`
>Shows general details about the specified nodes
---

 `chef generate cookbook cookbook/path/cookbook-name`
 >Generates a cookbook file. Path is optional, without it the cookbook will be created in the current working directory.

 `chef generate recipe recipe-name`
 >Generates a new recipe file

 *Recipes can be written and referenced in the default.rb file (created in every recipe file). When referencing the file in default.rb, format it like so:<br/> `include recipe recipe-folder-in-cookbook::recipe-file`*
 >If you have a cookbook called create_user, and you made a file called user.rb, you would write this as `include recipe create_user::user`

`cookstyle cookbook-file.rb`
>Checks for proper syntax

`knife cookbook upload recipe-directory,more-recipes-comma-separated`
>Uploads your recipe to the Chef Server

`knife cookbook list`
>List the cookbooks currently on the Chef Server

`chef-run node-name recipe`
>Runs a cookbook, as long as the knife.rb indicates the correct directory that the cookbooks are stored in. If not, you can substitute `recipe` for `path/to/recipe. Works best if run from the cookbooks directory.`

`chef-client --local-mode --why-run recipe`
>Performs a smoke test to check that we're getting the expected output. Only use --local mode when using Chef Zero. Otherwise run line without --local-mode. To apply changes, remove --why-run, because --why-run is used to check the output without applying it.

## **Docker**
`sudo apt install docker.io`
>Installs Docker for CLI on Linux ubuntu.

`sudo usermod -aG docker $USER`
>Adds the current user to the Docker group. Follow up with the command below to avoid a common daemonize error

`newgrp docker`
>Logs in to docker group. If a daemon issue persists beyond the previous two steps, you may need to reboot your system.

## **Elastic Stack**
**Installing**
>To see the most recent versions, visit [the Elastic website](https://www.elastic.co/start)

`wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.0.0-linux-x86_64.tar.gz`
>Download the tar file for Elasticsearch

`wget https://artifacts.elastic.co/downloads/kibana/kibana-8.0.0-linux-x86_64.tar.gz`
>Download the tar file for Kibana

`tar -zxf elasticsearch*.tar.gz`
`tar -zxf kibana*.tar.gz`
>Installs elasticsearch and kibana from the previously downloaded tar files

`ssh user@xxx.xx.xx.xx -L 5601:localhost:5601`
>When attempting to open Kibana in a browser while Kibana is running in a VM or Cloud environment, enter this command in a local terminal and it will forward the 5601 port to your localhost:5601

`bin/elasticsearch-users useradd my_admin -p my_password -r superuser`
>Creates a new user/password to access Elastic resources. Use of the superuser flag is optional.

## **Git**
```
git config --global user.name perscholas2021
git config --global user.email perscholas2021@gmail.com
```
>Configures your git username and email to identify who is making the commits

`git branch -m master main`
>Changes name of git branch from master to main. Replace `master` and `main` to rename branches as necessary

`git remote add origin repo-link`
>First time setting the remote repo

`git remote set-url origin repo-link`
>Replace an existing remote Link

`git remote -v`
>Check the remote repository linked to the local repository.

---
`git branch -m master main`
>Changes the branch name from master to main. If you're changing branch name mid-project, use the following 3 steps to update branch name everywhere:

`git push -u origin main`
>Pushes the main branch to the remote repository to prep for master deletion

`git symbolic-ref refs/remotes/origin/HEAD refs/remotes/origin/main`
>Points HEAD to main branch.

`git push origin --delete master`
>Deletes the master branch from the remote repository.
---

## **Jenkins**

`java -jar jenkins.war`
>Installs Jenkins at the default port 8080 from the .war file offered on (jenkins.io). *Optional: add --httpPort=xxxx to assign Jenkins to a specific port number other than 8080*

## **Kubernetes**

**Basic yaml orchestration format**
```
apiVersion: (v1 for Pod, apps/v1 for others listed below)
kind: (Pod, ReplicationController, ReplicaSet, Deployment)
metadata:
  name: name-here
  labels:
    labelKey: labelValue
    labelKey2: anotherLabelValue
spec:  
```

>If using ReplicaSet, ReplicationController, or Deployment, 'template:'' must be within 'spec:', as shown below  
```
spec:
  template:
    metadata:
      name: name-of-pods
      labels:
        key: value
    spec:
      containers:
      - name: container-name
        image: dockerhub-image
        ports:
        - containerPort: portNum to expose from node
```
>At the end of a Deployment or ReplicationController yaml file, you can control
the number of replicas by adding "replicas:" in line with template:
If using ReplicaSet or Deployment, you can match pod control up with other pods
that have matching labels by adding the following to the end of the file:
selector:
  matchLabels:
    key: value

**Yaml service file format**
```
apiVersion: v1
kind: Service
metadata:
  name: app-name
spec:
  type: NodePort
  ports:
    - targetPort: 80 (or any other port num. If not included, num will match port below)
      port: 80 (or any other port. Must be included)
      nodePort: between 30000 - 32767
  selector:
    (add labels here that match labels of pods you want to connect to)
```

`kubectl create -f yaml-file.yaml`
>Creates a deployment based on the details outline in the yaml file. If kind: Service
is specified in the yaml file, it will launch services to expose pods.

`kubectl create deployment --image=image-name deployment-name`
>Creates a deployment from an image without the need for a yaml file. Add the --replicas flag to specify the scale to launch at.

`kubectl run pod-name --image=image-to-pull-from-dockerhub`
>Creates a pod out of an image pulled from Docker Hub. Add --dry-run=client -o yaml to create a detailed yaml file that can be modified later with namespaces, scale specs, etc.

`kubectl apply -f file.yaml`
>Refreshes a pod with the newest image and pod details

`kubectl replace -f yaml-file.yaml`

`kubectl scale --replicas=x yaml-file.yaml`
>Scales amount of pod reps

`kubectl edit replicaset replica-set-name`
>Allows the user to edit the yaml file that was created by kubernetes in memory based on the initial yaml file. This does not change the original yaml file but allows changes to be made to the pod environment, such as replica scaling.

`kubectl describe replicaset replica-set-name`
>Displays a detailed description of the replica set status

`kubectl get all`
>Displays all pods, replica set, deployments, etc. at once.

`kubectl get namespaces`
>Displays a list of available namespaces.

`kubectl get pods --all-namespaces`
>Shows all the pods available in all namespaces. May swap out --all-namespaces with a specific namespace.

`minikube service service-name --url`
>Displays the url and port num of the pod exposure in local node

`kubectl rollout status deployment/name-of-deployment`
>Displays a live status update of pod deployments

`kubectl rollout history deployment/name-of-deployment`
>Displays the deployment update history with reason for update

`kubectl rollout undo deployment/name-of-deployment`
>Rolls back a deployment to the previous version

`kubectl proxy --address 0.0.0.0 --accept-hosts 192.168.1.XXX`
>Exposes service connection to ip address specified at --accept-hosts

>Proxy must be accessed via this link format:<br />
http://192.168.1.xxx:8001/api/v1/namespaces/default/services/service-name:XXXX/proxy/<br />
Use --accept-hosts ip with port 8001, fill in service-name with the name of your deployment's service, and use the port of the service itself (not targetPort or nodePort, just port)

## **MySQL**
`CREATE DATABASE <name>;`
>Creates a SQL database.

`DROP DATABASE <name>;`
>Deletes the selected database. DROP may be use for other items such as tables.

`SHOW DATABASES;`
>Shows all available databases.

`USE <database>`;
>Moves into a database for use.

`SELECT database();`
>Displays the database currently being used.

```
CREATE TABLE <tablename>
(
  <column_name> <data_type>,
  name varchar(255),
  age int
);
```
>Creates a table with a given name and defines the columns to be in that table. The varchar data type is for strings, and can't be set beyond 255, but does not have to be set to the maximum range. Does not have to span multiple lines. Don't forget the commas!

`SHOW TABLES`
>Shows the tables available within a database.

`SHOW COLUMNS FROM <table>;`
>Displays the columns available in a table.

`DROP TABLE <table>;`
>Deletes the selected table.

```
INSERT INTO <tablename>(column_name1, column_name2)
VALUES ("This data goes in column1", 777);
```
>Insert data into columns by name. There is a positional relationship as to what data is applied to which column i.e. 777 would be applied to column_name2 in this example. More than one set of values may be added at a time by adding more sets of information after VALUES, within parentheses, and with commas separating each set of data.

`SELECT * FROM <tablename>;`
>Displays all of the info that has already been inserted into the table.

`SELECT * FROM <table1>, <table2> WHERE <table1>.date=<table2>.date`
>This is a table join statement. This combines data from table1 and table2 based on the conditions defined in the WHERE statement. In this case the search displays all the cases where a date from table1 matches a date from table2.

## **Puppet**
[Official Site](https://puppet.com/docs/puppet/7/install_puppet.html#install_puppet)

**Puppet Server Installation (Ubuntu)**
---
`wget https://apt.puppet.com/puppet7-release-focal.deb`
>Downloads the Puppet package repositories for installation.

`dpkg -i <FILE_NAME>.deb`
>Installs the Puppet files from the previous step.

`apt-get install puppetserver`
>Installs the Puppet Server software.

`systemctl start puppetserver`
>Starts the Puppet Server as a service in Ubuntu.

`bash -l`
>Update PATH
---

**Puppet Agent Installation (Ubuntu)**
---
`wget https://apt.puppet.com/puppet7-release-focal.deb`
>Downloads the Puppet package repositories for installation.

`dpkg -i <FILE_NAME>.deb`
>Installs the Puppet files from the previous step.

`sudo apt-get install puppet-agent`
>Installs the Puppet agent software.

`sudo /opt/puppetlabs/bin/puppet resource service puppet ensure=running enable=true`
>Start the Puppet service.

`source /etc/profile.d/puppet-agent.sh`
OR
`export PATH=/opt/puppetlabs/bin:$PATH`
>Adds the puppet command to PATH. The "source" method uses a script installed by puppet-agent to add to PATH.
---

`/etc/default/puppetserver`
>Contains init settings for Puppet server in Ubuntu.

`puppet agent -t --config ./temporary_config.conf`
>"The puppet.conf file is always located at $confdir/puppet.conf. Although its location is configurable with the config setting, it can be set only on the command line." Above is an example usage. [Ref](https://puppet.com/docs/puppet/latest/config_file_main.html)

## **Python**

#### Install Pytest
[Reference](https://pytest-flask.readthedocs.io/en/latest/tutorial.html)

#### Using Pytest
[Reference](https://iammehdi.medium.com/testing-flask-apps-with-pytest-5b7af093c53d)

## **Ruby**

`puts` vs `print`
>Puts will automatically input a new line at the end of the string. Print requires a "\n" line break for any new lines.

## **Terraform**
**AWS EC2 Provision**
```
provider "aws" {
  region = "us-east-2"
  access_key = "pasteAccessKeyFromIAM"
  secret_key = "pasteSecretKeyFromIAM"
}

resource "aws_instance" "instance-name" {
  ami = "ami-amiCodeShownWhenSelectingImage"
  instance_type = "t2.micro"
}
```

`terraform fmt`
>Checks for proper formatting in configuration files and returns the names of corrected files. No output means there were no errors.

`terraform init`
>Initializes Terraform resources such as a state file. Perform from the same directory that your .tf file is in.

`terraform plan`
>Displays the changes that will occur when you apply your .tf file.

`terraform apply`
>Executes provisioning outlined in your .tf file.

`terraform destroy`
>Destroys items specified in the .tf file. If you want to only destroy specific items, add `-target <resource>` (Destroying the item in my demo .tf would be written `terraform destroy -target aws_instance.instance-name`)

`terraform refresh`
>Grabs the current state of your resources and updates the .tfstate file accordingly. Useful if resources are modified outside of Terraform and user wants to realign their resources with the desired state outlined in the .tf file.

`terraform state ls`
>Shows a list of all resources

`terraform state show <resource>`
>Shows a specified resource's state. Not including a specified resource will dump info about all of the resources.

# **Linux Systems**
>References to Linux commands that I've found useful along the way.

## **CentOS Terminal**

`rpm -ivh`
>RPM is the CentOS Package Manager. The -i flag indicates install packages, -v indicates verbose, and -h prints hash marks as package installs.

## **Ubuntu Terminal**

`sudo visudo`
>Add sudoers/change sudo privileges. Lets user edit /etc/sudoers. Add "<username> ALL=(ALL:ALL)"
Follow (ALL:ALL) with password settings - ALL is password enabled, NOPASSWD:
ALL disables password required when sudoing

`cat /etc/passwd`
>Gets general info about users

`scp <options> user@target_hostname_or_ip:absolute/path/files   <folder_local_system>`
>Copies files from a remote system to your local system. User can also switch <folder_local_system> and remote target to send files to a remote machine from local. Great way to transfer public keys.

`sudo swapon -a`
>Turns swap memory on

`sudo swapoff -a`
>Turns swap memory off
