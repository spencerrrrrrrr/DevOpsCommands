# Significant Code Lines

### **Linux Terminal**

`sudo visudo`
>Lets user edit /etc/sudoers. Add "<username> ALL=(ALL:ALL)"
Follow (ALL:ALL) with password settings - ALL is password enabled, NOPASSWD:
ALL disables password required when sudoing

### **Kubernetes**

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

`kubectl rollout history deployment/name-of-deployment`

`kubectl rollout undo deployment/name-of-deployment`

### **Ansible**
***Target users must have sudoer privileges in /etc/sudoers/
ansible_sudo_pass="yourPassword" >Place in inventory.txt file
ansible_host=192.168.x.x >Place in inventory.txt file. Assigns IP to target name
ansible_connection=ssh >Establishes connection type, "ssh" is Linux-based***
```
apt:
  pkg:
    python
    apache
    etc.
```
>Format to install packages in ubuntu via Ansible



### **Python**

##### Install Pytest
[Reference](https://pytest-flask.readthedocs.io/en/latest/tutorial.html)

##### Using Pytest
[Reference](https://iammehdi.medium.com/testing-flask-apps-with-pytest-5b7af093c53d)
