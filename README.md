# Kubernetes Cluster Setup with Vagrant

Deploy Nginx service under Kubernetes cluster with 3 Vagrant machines having following 
- 1 master node (k8s-master)
- 2 worker node (node1, node2)

Each machine is having 2048 GB memory and 2 CPUs (Check Vagrantfile)

Spin up the Vagrant hosts with following command

```
vagrant up
```

SSH to vagrant hosts
```
vagrant ssh << hostname >>
```
Hostname in our case: k8s-master, node1 and node2
