# Kubernetes Cluster Setup with Vagrant

Deploy Nginx service under Kubernetes cluster with 3 Vagrant machines having following 
- 1 master node (k8s-master)
- 2 worker node (node1, node2)

Each machine is having 2048 GB memory and 2 CPUs (check Vagrantfile)

### Start the hosts with following commands

```
vagrant up
```

### SSH to vagrant hosts
```
vagrant ssh << hostname >>
```

Hostname in our case: k8s-master, node1 and node2

### Update the packages on Ubuntu 16.04

```
apt update
```

### Setup Hosts

Edit hosts file on ALL server under /etc/hosts. 

```
192.168.50.10     k8s-master
192.168.50.11     node1
192.168.50.12     node2
```

Once done, test ping ALL servers hostname and make sure all IP address get resolved as a hostname

```
ping k8s-master
ping node1
ping node2

```
### Docker Installation

Install Docker from the Ubuntu repository

```
sudo apt install docker.io -y
```

Start and enable docker service to launch everytime at system boot

```
sudo systemctl start docker
sudo systemctl enable docker
```

### Disable SWAP 

In order to setup Kubernetes Linux server, it's required to disable the SWAP. Check the SWAP and disable it

```
sudo swapon -s
sudo swapoff -a
```

To disable the SWAP permanently, edit /etc/fstab file by commenting the SWAP partition type and ***reboot*** the system
```
#/dev/mapper/vagrant--vg-swap_1 none            swap    sw              0       0
```

### Install kubeadm, kubelet and kubectl

**kubeadm:** *the command to bootstrap the cluster.*

**kubelet:** *the component that runs on all of the machines in your cluster and does things like starting pods and containers.*

**kubectl:** *the command line util to talk to your cluster.*

Ref: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

Install apt-transport-https

```
sudo apt install -y apt-transport-https
```

Add the Kubernetes Key

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

Add the Kubernetes repository under /etc/apt/sources.list.d/

```
vi kubernetes.list
```

Paste below repository
```
deb http://apt.kubernetes.io/ kubernetes-xenial main
```

Update the repository and install packages
```
sudo apt update
sudo apt install -y kubeadm kubectl kubelet
```

### Kubernetes Cluster Initialization

Initialize the Kubernetes Cluster using Kubeadm command

```
sudo kubeadm init --pod-network-cidr=10.244.10.0/16 --apiserver-advertise-address=192.168.50.10 --kubernetes-version "1.17.0"
```

***--apiserver-advertise-address*** = *determines which IP address Kubernetes should advertise its API server on.*
***--pod-network-cidr*** = *specify the range of IP addresses for the pod network. We're using the 'flannel' virtual network. If you want to use another pod network such as weave-net or calico, change the range IP address.*


To avoid following warning while running pre-flight checks
```
  [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
```

Edit docker executable under ***/lib/systemd/system/docker.service***, look for the line ***ExecStart*** and add ***--exec-opt native.cgroupdriver=systemd***

```
....
....
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --exec-opt native.cgroupdriver=systemd
....
....
```

Reload the daemon and stop-start Docker service

```
systemctl daemon-reload
systemctl stop docker
systemctl start docker
```

In order to use the Kubenetes, require to run commands from the *kubeadm init...* result command

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
