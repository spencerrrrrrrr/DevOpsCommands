# Significant Code Lines
[GitHub Flavored Markdown Cheat Sheet](https://gist.github.com/stevenyap/7038119)
>Format contributions to GitHub Flavored Markdown

## **AWS CLI/CloudShell**
>If a command does not work, check that you have the appropriate permissions to perform the command and that your AWS CLI is up to date.

`aws iam list-users`
>Lists all the users that have been created in IAM.

## **Jenkins**

`java -jar jenkins.war`
>Installs Jenkins at the default port 8080 from the .war file offered on (jenkins.io). *Optional: add --httpPort=xxxx to assign Jenkins to a specific port number other than 8080*

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
is specified in the yaml file, it will launch services to expose pods

`kubectl run pod-name --image image-to-pull-from-dockerhub`
>Creates a pod out of an image pulled from Docker Hub

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

## **ELK Stack**
**Installing**
>To see the most recent versions, visit [the Elastic website](https://www.elastic.co/start)

`wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.0.0-linux-x86_64.tar.gz`
>Download the tar file for Elasticsearch

`wget https://artifacts.elastic.co/downloads/kibana/kibana-8.0.0-linux-x86_64.tar.gz`
>Download the tar file for Kibana

`tar -zxf <elasticsearch.tar file>`
`tar -zxf <kibana.tar file>`
>Installs elasticsearch and kibana from the previously downloaded tar files

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

## **Ruby**

`puts` vs `print`
>Puts will automatically input a new line at the end of the string. Print requires a "\n" line break for any new lines.


## **Python**

#### Install Pytest
[Reference](https://pytest-flask.readthedocs.io/en/latest/tutorial.html)

#### Using Pytest
[Reference](https://iammehdi.medium.com/testing-flask-apps-with-pytest-5b7af093c53d)

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
