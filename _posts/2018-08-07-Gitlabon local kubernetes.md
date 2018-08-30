---
layout: post
category: Kubernetes
title: Creating sinple Gitlab instance on Kubernetes Locally
tagline: by Henry
tags:
  - Gitlab
published: true
---

# Creating sinple Gitlab instance on Kubernetes Locally

__TAGS__
How to create a simple gitlab instance on Local Kubernetes
How to create a simple source code repository on a Local Kubernetes installation
How to attach to a kubernetes Pod
Gitlab email setttings with SendGrid
Example namespace use
Example kubernetes Deployment
Example Kubernetes Persistent Volume and Persistent Volume Claim use on a LOCAL installation
Example kubernetes Service


### Link to gist with full yaml
https://gist.github.com/henryza/1f5c6c022c02d641aaf2056a69f3c606

## Create Namespace for Gitlab
---
kind: Namespace
apiVersion: v1
metadata:
  name: gitlab
  labels:
    name: gitlab
	
## Create storage class (If one does not exist)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: Local
parameters:
  type: pd-standard
  
## Create a Persistent volume for Gitlab
kind: PersistentVolume
apiVersion: v1
metadata:
  name: gitlab-home
  labels:
    type: local
  namespace: gitlab
spec:
  storageClassName: slow
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/disks/gitlab
	
	
## Create a persistent volume claim for Gitlab
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: gitlab-home
  namespace: gitlab
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: slow
  resources:
    requests:
      storage: 5Gi
	  
## Create deployment for Gitlab
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitlab
  labels:
    app: gitlab
  namespace: gitlab
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gitlab
  template:
    metadata:
      labels:
        app: gitlab
    spec:
      volumes:
        - name: gitlab-home
          persistentVolumeClaim:
            claimName: gitlab-home
      containers:
      - name: gitlab
        image: gitlab/gitlab-ce
        imagePullPolicy: Always
        ports:
        - containerPort: 80
        - containerPort: 443
        - containerPort: 22
        volumeMounts:
          - mountPath: /etc/gitlab
            name: gitlab-home
			
## Create a service to expose Gitlab
apiVersion: v1
kind: Service
metadata:
  labels:
    app: gitlab
  name: gitlab
  namespace: gitlab
spec:
  ports:
  - name: gitlab-http
    port: 80
    protocol: TCP
    targetPort: 80
  - name: gitlab-https
    port: 443
    protocol: TCP
    targetPort: 443
  - name: gitlab-sshac
    port: 22
    protocol: TCP
    targetPort: 22
  selector:
    app: gitlab
  type: NodePort
  
## Find out the exposed Nodeport number for Gitlab
```kubectl get service | grep docker-registry```

__NOTE__
When using NodePort, the first port is internal and can only be accessed inside the kubernetes cluster using the internal IP address allocated to the service
The second port is for external access, using the ClusterIP which can be found using command
``` kubectl cluster-info ```
  
## Setting up the Gitlab instance
- get the pod name from gitlab namespace
 - ```kubectl get pod -n gitlab```
- attach to bash on the pod
 - ```kubectl exec -it PODNAME -n gitlab -- /bin/bash```
 
__GITLAB FILE__: /etc/gitlab/gitlab.rb
- Set the external URL
 ```external_url 'http://YOURURL'```
- Enable large files
```gitlab_rails['lfs_enabled'] = true```
- setup SMTP using Sendgrid
```
 gitlab_rails['smtp_enable'] = true
 gitlab_rails['smtp_address'] = "smtp.sendgrid.net"
 gitlab_rails['smtp_port'] = 465
 gitlab_rails['smtp_user_name'] = "apikey"
 gitlab_rails['smtp_password'] = "YOURPASSWORD"
 gitlab_rails['smtp_domain'] = "smtp.sendgrid.net"
 gitlab_rails['smtp_authentication'] = "login"
 gitlab_rails['smtp_enable_starttls_auto'] = true
 gitlab_rails['smtp_openssl_verify_mode'] = 'peer'
 gitlab_rails['gitlab_email_from'] = 'gitlab@YOURDOMAIN'
 gitlab_rails['gitlab_email_reply_to'] = 'noreply@zone.bcxcloud.com'

```
- Reconfigure gitlab to use the settings you provided
```gitlab-ctl reconfigure```