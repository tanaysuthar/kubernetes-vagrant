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
Deploy the ***flannel network*** to the kubernetes cluster using *kubectl* command

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Check after a minute to see nodes and pods
```
kubectl get nodes
kubectl get pods --all-namespaces
```

### Adding Worker nodes to kubernetes Cluster

Create a new token using kubeadm. By using the –print-join-command argument kubeadm will output the token and SHA hash required to securely communicate with the master

```
kubeadm token create --print-join-command
```

Paste the kubeadm join command to both the worker nodes
```
kubeadm join 192.168.50.10:6443 --token ucqvm7.9agh2kb8rlpqj41a     --discovery-token-ca-cert-hash sha256:30662c63788ee553c670b179acb3ac9e4b7eb59f4888e48ac63b4ff736a867b2 
```

Wait for some minutes and back to the 'k8s-master' node and check the node status

```
kubectl get nodes
```

### Deploy Nginx web server on Kubernetes Cluster

Create a new directory named 'nginx' and go to that directory

```
mkdir -p nginx/
cd nginx/
```

Create Nginx deployment file ***nginx-deployent.yaml*** with following configurations 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.0
        ports:
        - containerPort: 80
```

Create nginx deployment using ***kubectl*** command

```
kubectl create -f nginx-deployment.yaml
```

Check the deployment list inside the cluster
```
kubectl get deployments

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           93s

```

Check the kubernetes Pods and details of individual pod

```
kubectl get pods

NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7f555c9b4b-544s7   1/1     Running   0          79s
nginx-deployment-7f555c9b4b-696s5   1/1     Running   0          79s
nginx-deployment-7f555c9b4b-mc88k   1/1     Running   0          79s
```
