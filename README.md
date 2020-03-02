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
