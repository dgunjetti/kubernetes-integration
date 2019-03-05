# Kubernetes + cert-manager + ingress + letsencrypt = https

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
```

## Deploy a example service
```
$ kubectl create deployment nginx --image=nginx
$ kubectl expose deployment nginx --port=80
```

