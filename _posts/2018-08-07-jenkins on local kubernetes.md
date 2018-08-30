---
layout: post
category: Kubernetes
title: Creating a simple Docker Registry on a Local Kubernetes installation
tagline: by Henry
tags:
  - Jenkins
published: true
---

# Creating a simple Docker Registry on a Local Kubernetes installation

__TAGS__
Get http://localhost:5000/v2/: net/http: request canceled (Client.Timeout exceeded while awaiting headers
How to create a simple Jenkins installation on Kubernetes
Simple Jenkins container setup
Example kubernetes Deployment
Example Kubernetes Persistent Volume and Persistent Volume Claim use on a LOCAL installation
Example kubernetes Service

### Link to gist with full yaml
https://gist.github.com/henryza/10ff3a8470f53602c2251f31cbe76ec1

## Create Namespace for Jenkins
---
kind: Namespace
apiVersion: v1
metadata:
  name: jenkins
  labels:
    name: jenkins
	
## Create storage class (If one does not exist)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: Local
parameters:
  type: pd-standard
  
## Create a Persistent volume for Jenkins
kind: PersistentVolume
apiVersion: v1
metadata:
  name: jenkins-home
  labels:
    type: local
  namespace: jenkins
spec:
  storageClassName: slow
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/disks/jenkins/jenkins-home
	
## Create a Persistent volume claim for jenkins
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-home
  namespace: jenkins
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: slow
  resources:
    requests:
      storage: 5Gi
	  
## Create deployment for Jenkins
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  labels:
    app: jenkins
  namespace: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      volumes:
        - name: jenkins-home
          persistentVolumeClaim:
            claimName: jenkins-home
      containers:
      - name: jenkins
        image: jenkins/jenkins
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
        - containerPort: 5000
        volumeMounts:
          - mountPath: /var/jenkins_home
            name: jenkins-home
			
## Create a service to expose Jenkins
apiVersion: v1
kind: Service
metadata:
  labels:
    app: jenkins
  name: jenkins
  namespace: jenkins
spec:
  ports:
  - name: jenkins-reg
    port: 5000
    protocol: TCP
    targetPort: 5000
  - name: jenkins-http
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: jenkins
  type: NodePort
  
## Find out the exposed Nodeport number for Jenkins
```kubectl get service | grep docker-registry```

__NOTE__
When using NodePort, the first port is internal and can only be accessed inside the kubernetes cluster using the internal IP address allocated to the service
The second port is for external access, using the ClusterIP which can be found using command
``` kubectl cluster-info ```

## Setting up the Jenkins instance
- get the pod name from gitlab namespace
 ```kubectl get pod -n jenkins```
- attach to bash on the pod
 ```kubectl exec -it PODNAME -n jenkins -- /bin/bash```
- get your admin password
 ```cat /var/jenkins_home/secrets/initialAdminPassword```
- Copy this and browse to the Jenkins through the ClusterIP and Nodeport, use this password for your initial Jenkins setup

