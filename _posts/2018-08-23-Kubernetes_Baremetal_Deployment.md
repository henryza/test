---
layout: post
category: sites
title: How to install Kubernetes on Centos Bare Metal
tagline: by Henry
tags:
  - Kubernetes
  - Centos
published: true
---

# This is a file that I used to recently install Kubernetes on Bare Metal

## Master Kubernetes server

### Update and install screen
```
yum update -y && yum install screen -y
yum clean all
```

__NOTE__: Screen is there to help you detach a session incase you get kicked off the server.
 [Screen Documentation](https://centoshelp.org/resources/scripts-tools/a-basic-understanding-of-screen-on-centos/)

 ### Setup Network
 __NOTE__: This can either be done via nmtui or file
```
cat <<EOF > /etc/sysconfig/network-scripts/ifcfg-ens192
TYPE=Ethernet
BOOTPROTO=none
DEFROUTE=yes
IPV6INIT=no
NAME=ens192
DEVICE=ens192
ONBOOT=yes
IPADDR=192.168.0.1 1
PREFIX=24
GATEWAY=192.168.0.1
DNS1=8.8.8.8
EOF
```

### Disable Firewall and Network Manager (Unless you want to configure firewall)
```
systemctl restart network NetworkManager
systemctl stop NetworkManager
systemctl disable NetworkManager
systemctl stop firewalld
systemctl disable firewalld
```

### Setup hosts file (All your servers)
```
echo "192.168.0.10 kubmaster" >>  /etc/hosts
echo "192.168.0.11 kubworker01" >> /etc/hosts
```

### Setup requirements for kubernetes
```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
setenforce 0
swapoff -a
sed -i 's/\/dev/#\/dev/g' /etc/fstab
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
```

### Install Docker-CE
```
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
__NOTE__: As kubernetes has not moved to the latest docker version, you require to run a different package
```
yum install docker-ce-selinux -y
```

### Installing kubernetes
Creating the repo
```
cat <<ECHO > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

Updating the repo then installing kuberneres
```
yum install -y kubelet kubeadm kubectl
sed -i 's/cgroup-driver=systemd/cgroup-driver=cgroupfs/g' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
systemctl daemon-reload
systemctl restart docker && systemctl enable docker
systemctl restart kubelet && systemctl enable kubelet
```

### Initilizing Master
```
kubeadm init --pod-network-cidr 10.244.0.0/16
```



TO view any event you use describe method
EXAMPLE: kubectl describe pod hello-world

### OUTPUT from Initilizing Master

```
To start using your cluster, you need to run the following as a regular user:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 192.168.0.10:6443 --token 2wqgxah.ezyqhcfetti0zbsl --discovery-token-ca-cert-hash sha256:hfoihewihfioehiojfhewifhofhwoihefoihewifhoiewhfoihew
```

### Create environment variables
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Install Flannel Network
```
kubectl apply -f  https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Check if Flannel comes up
```
watch -n 1 kubectl get pods --all-namespaces
 ```

## Dashboard (Not Reccomended)
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml

To access the dashboard you use (on the box) Won't work because no GUI is installed
kubectl proxy

http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```


## Install cockpit
ACCESS ON :https://hostname:9090/
```
yum install cockpit cockpit-dashboard cockpit-docker cockpit-kubernetes -y
firewall-cmd --add-port=9090/tcp
firewall-cmd --permanent --add-port=9090/tcp
systemctl enable cockpit.socket
systemctl start cockpit.socket
systemctl restart cockpit.socket
```

# Workert Node
## Install cockpit
ACCESS ON :https://hostname:9090/
```
yum install cockpit cockpit-dashboard cockpit-docker cockpit-kubernetes -y
firewall-cmd --add-port=9090/tcp
firewall-cmd --permanent --add-port=9090/tcp
systemctl enable cockpit.socket
systemctl start cockpit.socket
systemctl restart cockpit.socket
```

### Update and install screen
```
yum update -y && yum install screen -y
yum clean all
```

__NOTE__: Screen is there to help you detach a session incase you get kicked off the server.
 [Screen Documentation](https://centoshelp.org/resources/scripts-tools/a-basic-understanding-of-screen-on-centos/)

 ### Setup Network
 __NOTE__: This can either be done via nmtui or file
```
cat <<EOF > /etc/sysconfig/network-scripts/ifcfg-ens192
TYPE=Ethernet
BOOTPROTO=none
DEFROUTE=yes
IPV6INIT=no
NAME=ens192
DEVICE=ens192
ONBOOT=yes
IPADDR=192.168.0.1 1
PREFIX=24
GATEWAY=192.168.0.1
DNS1=8.8.8.8
EOF
```

### Disable Firewall and Network Manager (Unless you want to configure firewall)
```
systemctl restart network NetworkManager
systemctl stop NetworkManager
systemctl disable NetworkManager
systemctl stop firewalld
systemctl disable firewalld
```

### Setup hosts file (All your servers)
```
echo "192.168.0.10 kubmaster" >>  /etc/hosts
echo "192.168.0.11 kubworker01" >> /etc/hosts
```

### Setup requirements for kubernetes
```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
setenforce 0
swapoff -a
sed -i 's/\/dev/#\/dev/g' /etc/fstab
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
```

### Install Docker-CE
```
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
__NOTE__: As kubernetes has not moved to the latest docker version, you require to run a different package
```
yum install docker-ce-selinux -y
```

### Installing kubernetes
Creating the repo
```
cat <<ECHO > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

Updating the repo then installing kuberneres
```
yum install -y kubelet kubeadm kubectl
sed -i 's/cgroup-driver=systemd/cgroup-driver=cgroupfs/g' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
systemctl daemon-reload
systemctl restart docker && systemctl enable docker
systemctl restart kubelet && systemctl enable kubelet
```

### Add worker to Cluster
```
kubeadm join 192.168.0.10:6443 --token 2wqgxah.ezyqhcfetti0zbsl --discovery-token-ca-cert-hash sha256:hfoihewihfioehiojfhewifhofhwoihefoihewifhoiewhfoihew
```

make sure it comes up, might take a while
```
watch -n 1 kubectl get nodes --all-namespaces
```


# Troubleshooting common ERRORS
- Kubernetes DNS will not start without Network component installed
- Worker node or master in state other than ready, or pods. do kubectl descripe pods / deployment / service. Most network issues are from above, or not using a /16 with flannel network
- The cidr has to be a /16 to work properly

## types
- type:LoadBalancer only works on clouds
- type:Nodeport works on local deployments, creates an open port mapping to resource on all nodes. - - First port is the (nat'd) port, second is the access port (80:30395/TCP)
- type:ingress is for http mappings

## Common Commands

```
# ALL PODS
 kubectl get pods --all-namespaces

# ALL SERVICES
 kubectl get service --all-namespaces

#ALL DEPLOYMENT
 kubectl get deployments --all-namespaces

#ALL PHYSICAL VOLUMES
 kubectl get pv --all-namespaces

#ALL PHYSICAL VOLUME CLAIMS
 kubectl get pvc --all-namespaces

#ALL namespaces
 kubectl get namespaces

#ALL SECRETS
 kubectl get secrets

#ALL CONFIG MAPS
 kubectl get configmap

#ALL INGRES CONTROLLERS
 kubectl get ingress

#ALL NODES
 kubectl get nodes --all-namespaces

#TO VIEW ERRORRS
 kubectl describe service/pod/deployment

#To edit an existing deployment
 kubectl edit deployment YOUDEPLOYMENTNAME
 save with :wq

# To Get Cluster info
  kubectl cluster-info

# To get a config of something already running (in yaml)
get pod mypod -o=yaml
```


## How to reset kubernetes correctly
```
kubeadm reset
systemctl stop kubelet
systemctl stop docker
rm -rf /var/lib/cni/
rm -rf /var/lib/kubelet/*
rm -rf /etc/cni/
ifconfig cni0 down
ifconfig flannel.1 down
ifconfig docker0 down
ip link delete cni0
ip link delete flannel.1
systemctl start kubelet
systemctl start docker
```

## How to remove the kubernetes dashboard
```
kubectl delete deployment kubernetes-dashboard --namespace=kube-system
kubectl delete service kubernetes-dashboard  --namespace=kube-system
kubectl delete role kubernetes-dashboard-minimal --namespace=kube-system
kubectl delete rolebinding kubernetes-dashboard-minimal --namespace=kube-system
kubectl delete sa kubernetes-dashboard --namespace=kube-system
kubectl delete secret kubernetes-dashboard-certs --namespace=kube-system
kubectl delete secret kubernetes-dashboard-key-holder --namespace=kube-system
```

# NEW INF TO SORT

## Cheatsheet
https://kubernetes.io/docs/reference/kubectl/cheatsheet/

## Autoscale
https://kubernetes.io/blog/2016/07/autoscaling-in-kubernetes/
kubectl autoscale deployment hello-kubernetes--cpu-percent=50 --min=1 --max=10
