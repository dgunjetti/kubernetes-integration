
Download v1.12.0 

sudo su
apt-get update && apt install docker.io -y

apt-get update && apt-get install -y apt-transport-https curl

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl

swapoff -a

kubeadm init --pod-network-cidr=192.168.0.0/16

kubeadm join 172.31.3.36:6443 --token <toke> --discovery-token-ca-cert-hash <discovery-token>

export KUBECONFIG=/etc/kubernetes/admin.conf

