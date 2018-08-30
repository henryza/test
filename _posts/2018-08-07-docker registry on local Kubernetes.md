---
layout: post
category: Kubernetes
title: Creating a simple Docker Registry on a Local Kubernetes installation
tagline: by Henry
tags:
  - Docker_Registry
published: true
---

# Creating a simple Docker Registry on a Local Kubernetes installation

__TAGS__
Get http://localhost:5000/v2/: net/http: request canceled (Client.Timeout exceeded while awaiting headers
How to setup insecure docker registry
How to setup docker registry on local kubernetes
Simple docker registry
Example kubernetes Deployment
Example Kubernetes Persistent Volume and Persistent Volume Claim use on a LOCAL installation
Example kubernetes Service

### Link to gist with full yaml
https://docs.docker.com/registry/insecure/#deploy-a-plain-http-registry

__Windows__
- Right click on Docker, go to Setitings, Daemon. 
- Click on the button to make it Advanced
- Insert the following into Daemon or ammend what is already there
 ```{ "insecure-registries" : ["yourregistry:5000"] }```
- Restart docker
ALTERNATIVELY
- Open file C:\Users\henry.stock\.docker\daemon.json
- Add a section, or amend if it exists to
	```{ "insecure-registries" : ["yourregistry:5000"] }```

__Linux__
- Create file /etc/docker/daemon.json
- Insert { "insecure-registries" : ["yourregistry:5000"] }
- Restart docker

## Create storage class (if one doesn't exist)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: Local
parameters:
  type: pd-standard

## Create a Persistent volume
kind: PersistentVolume
apiVersion: v1
metadata:
  name: docker-reg
  labels:
    type: local
spec:
  storageClassName: slow
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/disks/dockerRegistry" 
	
	
## Create a Persistent Volume Claim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: docker-reg
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: slow
  resources:
    requests:
      storage: 5Gi
	  
## Create a Deplyment for the docker registry
apiVersion: apps/v1
kind: Deployment
metadata:
  name: docker-registry
  labels:
    app: docker-registry
spec:
  replicas: 1
  selector:
    matchLabels:
      app: docker-registry
  template:
    metadata:
      labels:
        app: docker-registry
    spec:
      volumes:
        - name: registry-data
          persistentVolumeClaim:
            claimName: docker-reg
      containers:
      - name: registry
        image: registry:2
        imagePullPolicy: Always
        ports:
          - containerPort: 5000
        volumeMounts:
          - mountPath: "/var/lib/registry"
            name: registry-data
			
## Create a service to expose the docker registry
apiVersion: v1
kind: Service
metadata:
  labels:
    app: docker-registry
  name: docker-registry
  namespace: default
spec:
  clusterIP: 10.97.210.140
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 32421
    port: 5000
    protocol: TCP
    targetPort: 5000
  selector:
    app: docker-registry
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
  
## Find out the exposed Nodeport number for the docker registry service
```kubectl get service | grep docker-registry```

__NOTE__
When using NodePort, the first port is internal and can only be accessed inside the kubernetes cluster using the internal IP address allocated to the service
The second port is for external access, using the ClusterIP which can be found using command
``` kubectl cluster-info ```
