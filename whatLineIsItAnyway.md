#Significant Code Lines

###**Linux Terminal**

**sudo visudo**
>Lets user edit /etc/sudoers. Add "<username> ALL=(ALL:ALL)"
Follow (ALL:ALL) with password settings - ALL is password enabled, NOPASSWD:
ALL disables password required when sudoing

###**Kubernetes**
######**Basic yaml format**
>apiVersion: v1
kind: (Pod, ReplicationController, ReplicaSet, etc.)
metadata:
  name: name-here
  labels: (optional labels)
spec:
  Specify container details, what you want the container to be named, and
  which image you want the container to use

>If using ReplicaSet or ReplicationController, template: must be within spec:
i.e.  
spec:
  template:
    pod-specs: (metadata:, spec:)

>At the end of a ReplicationController yaml file, you can control the number of
replicas by adding "replicas:" in line with template:
If using ReplicaSet, you can match pod control up with other pods that have
matching labels by adding the following to the end of the file:
selector:
  matchLabels:
    key: value


**kubectl create -f yaml-file.yaml**
>Creates a deployment based on the details outline in the yaml file

**kubectl run pod-name --image image-to-pull-from-dockerhub**
>Creates a pod out of an image pulled from Docker Hub

**kubectl apply -f file.yaml**
>Refreshes a pod with the newest image and pod details

**kubectl replace -f yaml-file.yaml**

**kubectl scale --replicas=x yaml-file.yaml**
>Scales amount of pod reps

**kubectl edit replicaset replica-set-name**
>Allows the user to edit the yaml file that was created by kubernetes in memory
>based on the initial yaml file. This does not change the original yaml file but
>allows changes to be made to the pod environment, such as replica scaling.

**kubectl describe replicaset replica-set-name**
>Displays a detailed description of the replica set status

###**Ansible**
***Target users must have sudoer privileges in /etc/sudoers/
ansible_sudo_pass="yourPassword" >Place in inventory.txt file
ansible_host=192.168.x.x >Place in inventory.txt file. Assigns IP to target name
ansible_connection=ssh >Establishes connection type, "ssh" is Linux-based***
**apt:**
    **pkg:**
      **python**
      **apache**
      **etc.** >Format to install packages in ubuntu via Ansible



###**Python**

#####Install Pytest
[Reference](https://pytest-flask.readthedocs.io/en/latest/tutorial.html)

#####Using Pytest
[Reference](https://iammehdi.medium.com/testing-flask-apps-with-pytest-5b7af093c53d)
