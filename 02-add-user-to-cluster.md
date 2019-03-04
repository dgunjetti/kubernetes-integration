# Authenticate and Authorize user in Kubernetes

Following are steps to create authentication and role based access control for user.

## Generate certificate for user - Bob
```
openssl genrsa -out bob.key 2048
openssl req -new -key bob.key -out bob-csr.json -subj "/CN=bob"
```

## Create certificate signing request
```
cat <<EOF | kubectl create -f -
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: user-bob-csr
spec:
  groups:
  - system:authenticated
  request: $(cat bob-csr.json | base64 | tr -d '\n')
  usages:
  - digital signature
  - key encipherment
  - client auth
EOF
```

## Approve certificate as admin
```
kubectl get csr
kubectl certificate approve user-bob-csr
```

## Download Bob's assigned certificate
```
kubectl get csr user-bob-csr -o jsonpath='{.status.certificate}' \
    | base64 --decode > bob.crt
```

## Generate kubeconfig
In below command replace < master-nod-ip > with actual master node ip or dns name.
```
  kubectl config set-cluster test-cluster \
    --insecure-skip-tls-verify=true \
    --server=https://< master-nod-ip >:6443 \
    --kubeconfig=bob.kubeconfig

  kubectl config set-credentials bob \
    --client-certificate=bob.crt \
    --client-key=bob.key \
    --embed-certs=true \
    --kubeconfig=bob.kubeconfig

  kubectl config set-context default \
    --cluster=test-cluster \
    --user=bob \
    --kubeconfig=bob.kubeconfig

  kubectl config use-context default --kubeconfig=bob.kubeconfig
```

## Try to get pods information 
Using bob.kubeconfig try to get pod information
```
kubectl get pod --kubeconfig=bob.kubeconfig
Error from server (Forbidden)
```
The user Bob got authenticated but authorization is failing.

## Create pods view access permission for Bob

```
kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods
kubectl create clusterrolebinding read-pod --clusterrole=pod-reader --user=bob
```

## Try to get pods information 
```
kubectl get pod --kubeconfig=bob.kubeconfig
```
now list the pods should be visible

## Try to get nodes information
```
kubectl get nodes --kubeconfig=bob.kubeconfig
Error from server (Forbidden)
```
Since access is not provided to get nodes information, it should still fail.




