# Creating Kubernetes cluster using Kubeadm

Following are the steps for creating a kubernetes cluster with single master and worker node.

![](https://i.imgur.com/ToeuZEX.png)


## AWS EC2 Instance provisioning
Launch two t2-medium instances with Ubuntu 18.04 AMI.

## Edit Security Group 
Ports 6443 is opened for internal IP CIDR in security group. This port is used by worker node when joining to master.


## Following installations are done on both instances.

ssh into both instances.

### need root access
```
sudo su
```

### Install Docker
```
apt-get update && apt install docker.io -y
```

### Install kubeadm kubelet kubectl
```
apt-get update && apt-get install -y apt-transport-https curl

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt-get update apt-get install -y kubelet kubeadm 

kubectl apt-mark hold kubelet kubeadm kubectl
```

### Disable swap for kubelet to work properly.
```
swapoff -a
```


## Initialize kubeadm on master

We are using calico network plugin. It requires pod network to be in a specific IP CIDR. We need to specify that CIDR during kubeadm initialization.

```
kubeadm init --pod-network-cidr=192.168.0.0/16
```

kubeadm init will download and install cluster control plane components.

copy the entire 'kubeadm join' command giving at the end of kubeadm init output. It will be used to join worker node.

### export admin kubeconfig.
```
export KUBECONFIG=/etc/kubernetes/admin.conf
```

### Verify component status
```
kubectl get componentstatus
```

### install calico network plugin
```
kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```


## Add worker node
Run the copied 'kubeadm join' command.
```
kubeadm join 172.31.3.36:6443 --token <token> --discovery-token-ca-cert-hash <discovery-token>
```


## Verify on master

### Verify nodes are added
```
kubectl get nodes
```
It lists both master and worker node.

### Now cluster should be UP!!!



## Steps for tear down the cluster 

### running following commands from master
```
kubectl drain <worker node> --delete-local-data --force --ignore-daemonsets
kubectl delete node <worker node>

kubeadm reset

iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
```

### terminate the instances from AWS console.




