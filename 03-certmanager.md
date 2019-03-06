# Kubernetes + ingress + cert-manager + letsencrypt = https

## Install helm client
```
brew install kubernetes-helm
```

## Install tiller
Tiller is Helm's server-side component, which helm client uses to deploy resources. It is given admin privileges.
```
kubectl create serviceaccount tiller --namespace=kube-system
kubectl create clusterrolebinding tiller-admin --serviceaccount=kube-system:tiller --clusterrole=cluster-admin
helm init --service-account=tiller
```

## Update local helm repository charts
```
helm repo update
```

## Deploy nginx ingress controller
```
helm install stable/nginx-ingress --name=nginx-ingress
```
## Deploy cert-manager
```
kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.7/deploy/manifests/00-crds.yaml
helm install --name cert-manager --namespace cert-manager stable/cert-manager
```

## Configure Let's Encrypt Issuer
Edit the email address below

```
cat << EOF | kubectl apply -f -
apiVersion: certmanager.k8s.io/v1alpha1
kind: Issuer
metadata:
 name: letsencrypt-staging
spec:
 acme:
   # The ACME server URL
   server: https://acme-staging-v02.api.letsencrypt.org/directory
   # Email address used for ACME registration
   email: deepakgb79@gmail.com
   # Name of a secret used to store the ACME account private key
   privateKeySecretRef:
     name: letsencrypt-staging
   # Enable the HTTP-01 challenge provider
   http01: {}
EOF
```
```
cat << EOF | kubectl apply -f -
apiVersion: certmanager.k8s.io/v1alpha1
kind: Issuer
metadata:
 name: letsencrypt-prod
spec:
 acme:
   # The ACME server URL
   server: https://acme-v02.api.letsencrypt.org/directory
   # Email address used for ACME registration
   email: deepakgb79@gmail.com
   # Name of a secret used to store the ACME account private key
   privateKeySecretRef:
     name: letsencrypt-prod
   # Enable the HTTP-01 challenge provider
   http01: {}
EOF
```


## Deploy a nginx web server service
```
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --type=NodePort --port=80
```

## Create ingress resource
```
cat << EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: basic-ingress
spec:
  backend:
    serviceName: nginx
    servicePort: 80
EOF
```

## Get external ip address
```
kubectl get ingress basic-ingress
```
It will take some time for loadbalancer to get provisioned 

## point the browser to external ip
we may get HTTP 404 until external-ip is propogated.
we need to get 'welcome to nginx' html web page.


## create DNS entry mapping domain name with external ip address

## point the browser to dns name

## edit ingress resource to enable tls
Edit the hostname below
```
cat << EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: basic-ingress
  annotations:
    certmanager.k8s.io/issuer: "letsencrypt-prod"
    certmanager.k8s.io/acme-challenge-type: http01
spec:
  tls:
  - hosts:
    - test.hashfab.io
    secretName: test-hashfab-tls
  backend:
    serviceName: nginx
    servicePort: 80
EOF
```

## check certificate is created
```
kubectl get certificate
```

## update the dns entry with new external ip 
```
kubectl get ingress basic-ingress
```

## point the browser to dns name
wait for dns propogation.
we need to see web server is now available to https.



