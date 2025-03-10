## Pod to Pod Encryption
- You should be using Cilium as CNI
- there are two ways to do **Transparent Encryption**
    1) IPsec Transparent Encryption
    2) WireGuard Transparent Encryption


------ Goto: docs.cilium.io --------- Search: Transparent Encryption
------ Click: First link    --------- Read: Guide

- https://docs.cilium.io/en/stable/security/network/encryption/#gsg-encryption


```bash

# Check the encryption status
cilium encryption status
kubectl -n kube-system exec -it ds/cilium -- cilium encrypt status

# Lets do it

# Create secret
kubectl create -n kube-system secret generic cilium-ipsec-keys \
   --from-literal=keys="3+ rfc4106(gcm(aes)) $(echo $(dd if=/dev/urandom count=20 bs=1 2> /dev/null | xxd -p -c 64)) 128"

# Check the version
cilium status

cilium upgrade --version 1.17.0    --set encryption.enabled=true    --set encryption.type=ipsec

# Check the status
cilium encryption status
kubectl -n kube-system exec -it ds/cilium -- cilium encrypt status


```



