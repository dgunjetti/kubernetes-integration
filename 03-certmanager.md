# Kubernetes + ingress + cert-manager + letsencrypt = https

## Install helm client
```
$ brew install kubernetes-helm
```

## Install tiller
Tiller is Helm's server-side component, which helm client uses to deploy resources. It is given admin privileges.
```
$ kubectl create serviceaccount tiller --namespace=kube-system
$ kubectl create clusterrolebinding tiller-admin --serviceaccount=kube-system:tiller --clusterrole=cluster-admin
$ helm init --service-account=tiller
```

## Update local helm repository charts
```
$ helm repo update
```

## Deploy nginx ingress controller
```
$ helm install stable/nginx-ingress --name=nginx-ingress
$ kubectl get svc
```
check the external-ip for nginx-ingress-controller, it may take some time for cloud provider to provision loadbalancer
Create DNS entry mapping ingress controller external IP address with domain name

## Deploy a example service
```
$ kubectl create deployment nginx --image=nginx
$ kubectl expose deployment nginx --type=NodePort --port=80
```

