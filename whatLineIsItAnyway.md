# Significant Code Lines
[GitHub Flavored Markdown Cheat Sheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)
>Format contributions to GitHub Flavored Markdown

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

## **Git**
```
git config --global user.name perscholas2021
git config --global user.email perscholas2021@gmail.com
```
>Configures your git username and email to identify who is making the commits

`git branch -m master main`
>Changes name of git branch from master to main. Replace `master` and `main` to rename branches as necessary

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

**Setting Up Chef Server**
---
`sudo hostnamectl set-hostname chef-server.example.com`
>Set server hostname that will be the DNS name of your Chef Server

`sudo nano /ect/hosts`
`127.0.0.1 chef-server.example.com`
>Modifies access to your Chef host

```
sudo apt -y install chrony
sudo timedatectl set-timezone US/Michigani
sudo systemctl restart chrony
```
>Because Chef is prone to time drift, it's best to connect your system to Network Time Protocol (NTP)

```
VERSION="14.12.21
wget https://packages.chef.io/files/stable/chef-server/${VERSION}/ubuntu/18.04/chef-server-core_${VERSION}-1_amd64.deb
sudo apt install ./chef-server-core_${VERSION}-1_amd64.deb
```
>Installs Chef Server 2/1/22

`sudo chef-server-ctl user-create USER_NAME FIRST_NAME LAST_NAME EMAIL 'PASSWORD' --filename FILE_NAME`
>Creates an admin account and auto generates an rsa key. FILE_NAME is where the key will be stored

`sudo chef-server-ctl org-create short_name 'full_organization_name' --association_user user_name --filename ORGANIZATION-validator.pem`
>Creates an organization account. Name must begin with a lowercase letter or digit, no whitespace. user_name associates the specified user with the admins security group on the Chef server. --filename specifies where to save the rsa private key.

**Setting Up Chef Manage**
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
>Installs the Chef management console, **WHICH IS ONLY FREE FOR UP TO 25 NODES**

**Setting Up Chef Workstation**
---
`sudo ufw allow proto tcp from any to any port 80,443`
>Opens ports 80 and 443 because those are the ports Chef is using

```
VER="22.1.745"
wget https://packages.chef.io/files/stable/chef-workstation/${VER}/ubuntu/20.04/chef-workstation_${VER}-1_amd64.deb
```
>Installs Chef Workstation for Ubuntu 20.04 2/1/22
[Chef Workstation Downloads](https://www.chef.io/downloads/tools/workstation)

```
chef generate repo chef-repo
cd chef-repo
```
>Generates a Chef repo on your Workstation machine

`cookstyle recipe-file.rb`
>Checks for proper syntax

`chef-client --local-mode --why-run recipe-file.rb`
>Performs a smoke test to check that we're getting the expected output. Only use --local mode when using Chef Zero. Otherwise run line without --local-mode. To apply changes, remove --why-run, because --why-run is used to check the output without applying it.



## **Docker**
`sudo apt install docker.io`
>Installs Docker for CLI on Linux ubuntu

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

## **Ruby**

`puts` vs `print`
>Puts will automatically input a new line at the end of the string. Print requires a "\n" line break for any new lines.


## **Python**

#### Install Pytest
[Reference](https://pytest-flask.readthedocs.io/en/latest/tutorial.html)

#### Using Pytest
[Reference](https://iammehdi.medium.com/testing-flask-apps-with-pytest-5b7af093c53d)
