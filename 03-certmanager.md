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


## Deploy cert-manager
```
kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.6/deploy/manifests/00-crds.yaml
kubectl create namespace cert-manager
kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true
helm repo update
helm install \
  --name cert-manager \
  --namespace cert-manager \
  stable/cert-manager
```

## Verifying the cert-manager installation
```
kubectl get pods --namespace cert-manager
```

```
cat <<EOF > test-resources.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: cert-manager-test
---
apiVersion: certmanager.k8s.io/v1alpha1
kind: Issuer
metadata:
  name: test-selfsigned
  namespace: cert-manager-test
spec:
  selfSigned: {}
---
apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: selfsigned-cert
  namespace: cert-manager-test
spec:
  commonName: example.com
  secretName: selfsigned-cert-tls
  issuerRef:
    name: test-selfsigned
EOF
```

Create the test resources
```
kubectl apply -f test-resources.yaml
```

Check the status of the newly created certificate
```
kubectl describe certificate -n cert-manager-test
```

Clean up the test resources
```
kubectl delete -f test-resources.yaml
```

## Deploy a nginx web server service
```
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --type=NodePort --port=80
```

## Deploy nginx ingress controller
```
helm install stable/nginx-ingress --name=nginx-ingress
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

## Create DNS entry mapping domain name with external ip address

## Point the browser to dns name
Wait for dns propogation, it takes 5-10 minutes, we need to get 'welcome to nginx' html web page.

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

## Update ingress resource to enable tls
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

## Check certificate is created
```
kubectl get cert
```
wait for certificate status to become ready.


## Point the browser to https://\<dnsname\>
we need to see web server available on https.


## References
https://docs.cert-manager.io/en/latest/getting-started/install.html#

https://github.com/jetstack/cert-manager/blob/master/docs/tutorials/acme/quick-start/index.rst

