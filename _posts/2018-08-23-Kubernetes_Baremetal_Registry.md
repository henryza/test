---
layout: post
category: sites
title: Creating insecure Docker Registry on Base Metal
tagline: by Henry
tags:
  - Kubernetes
  - Centos
  - Registry
published: true
---

# Creating insecure Docker Registry on Base Metal

## Allowing for connecting to an insecure Registry
To enable this on the CLIENT side ([DOCUMENTATION](https://docs.docker.com/registry/insecure/#deploy-a-plain-http-registry))
- Windows
  - Right click on Docker, go to Setitings, Daemon.
  - Click on the button to make it Advanced
  - Insert the following into Daemon
  - { "insecure-registries" : ["yourregistry:5000"] }
  - Restart docker
  ![alt text](../assets/images/kubernetesWindows.JPG")

- Linux
  - Create file /etc/docker/daemon.json
  - Insert { "insecure-registries" : ["yourregistry:5000"] }
  - Restart docker


## Create storage class
A storage class is a good idea as normally on the cloud the PVC can auto gen from the storage class
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: Local
parameters:
  type: pd-standard
```

## Create a Persistent Volume
This created from the local disk. Make sure to provision the folder before else it might fail
```
## Physical volume
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
```


## Create Persistent Volume Claim
This allows the PV and Node communicate with the drive
```
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
```

## Deplyment
This will create a deployment with the PODS under it
```
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
```

## Service
This will expose out deployment to the outside
```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: docker-registry
  name: docker-registry
  namespace: default
spec:
  clusterIP: YOUR CLUSTER IP
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
```



__TROUBLESHOOTING__
These are the typical messages you get
- Get http://localhost:5000/v2/: net/http: request canceled (Client.Timeout exceeded while awaiting headers
  - This is due to the local docker insecureRegistry setting discussed as the first topic
- Repository Timeout
  - Make sure that you can access the clusterIP address of the pod on the specified port. Same for service

Tags:How to setup insecure docker registry, How to setup docker registry on kubernetes, Kubernetes docker registry
