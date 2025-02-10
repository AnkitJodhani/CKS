# Restrict API Server

## Access API SERVER using CURL
- Below is the command how you can hit apiserver endpoint with all the credentials
- like certificate authority, client certificate, client key
```bash
curl https://172.31.19.47:6443/api/v1/namespaces/kube-system/pods \
--cacert ca.crt --cert client.crt --key client.key

```


## Access API SERVER from local machine


1) Expose kubernetes svc as NodePort service from ClusterIP
```bash
k edit svc kubernetes -n default
```

2) Add WorkerNode IP entry in the /etc/hosts file
```bash
# IP-ADDRESS kubernetes
sudo vim /etc/hosts # (i.e 52.91.11.92 kubernetes)

# 🤔 BUT WHY
# Goto: master node ------> /etc/kubernetes/pki/
openssl x509  -noout -text -in apiserver.crt
# You will see the apiserver only allow below domains and Ips
# DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, DNS:master, IP Address:10.96.0.1, IP Address:172.31.27.167

```

2) Copy .kube/config from master to local machine
- Note: change the server URL to  server: https://kubernetes:ExposePort (i.e https://kubernetes:30390)


## NodeRestriction
- we want to restrict kubelet from modifying the node lable
- Coz there are some secure pod that needs to be run only some vey secure node & not anyother node
- Example:
    - Secure Nodes: Labeled as `role=secure` and equipped with enhanced security measures (e.g., hardware encryption, restricted access)
    - Regular Nodes: Labeled as role=regular with standard configurations
    - A critical application (e.g., financial transaction processor) is configured with a Node Selector to run only on nodes labeled `role=secure`
    - An attacker gains access to a regular node and attempts to modify its label from role=regular to `role=secure` using kubelet, tricking the scheduler into placing the sensitive pod on this compromised node
- To achive this you should have flag "- --enable-admission-plugins=NodeRestriction" in the /etc/kubernetes/manifests/kube-apiserver.yaml

```bash

# SSH into the worker node (worker-1)

# set KUBECONFIG environment variable or --kubeconfig path
# kubelet uses below file to talk to API SERVER
sudo -i # Might face permission issue - not able to read keys etc...
export KUBECONFIG=/etc/kubernetes/kubelet.conf
sudo kubectl get nodes --kubeconfig /etc/kubernetes/kubelet.conf


# You can't lable the anyother node
kubectl label node master temp=ok
kubectl label node worker-2 temp=ok

# User "system:node:worker-1" Can't list secrets - IT HAS LIMITED PERMISSION
kubectl get secrets

# You can lable yourself
kubectl label node worker-1 temp=ok

# There are some specific lable that you can't even assign to yourself
# for example: Prefix "node-restriction.kubernetes.io/"
# Example-1: example.com.node-restriction.kubernetes.io/fips=true
# Example-2: example.com.node-restriction.kubernetes.io/pci-dss=true

kubectl label node worker-1 example.com.node-restriction.kubernetes.io/fips=true
kubectl label node worker-1 node-restriction.kubernetes.io/secure=true

# SSH into master node and then you will be able to apply the labels
kubectl label node worker-1 example.com.node-restriction.kubernetes.io/fips=true
kubectl label node worker-1 node-restriction.kubernetes.io/secure=true


```
