## Encryption at rest
- We want to store data encrypted inside ETCD
- For more details checkout:
  - https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

## Lets create secret & attach to pod..
```bash
# Lets store secret files inside secret k8s secret

cat <<EOF > very-secure-files.yml
apiVersion: v1
kind: Secret
metadata:
  name: very-secure-files
data:
  .secretfile: dGhpcyBmaWxlcyBpcyB2ZXJ5IHZlcnkgc2VjdXJlIAoK
  myconfigfile: dGhpcyBmaWxlcyBoYXMgYWxsIAp0aGUgCm5lY2Vzc2FyeSAKY29uZmlndXJhaXRvbnMKCg==
EOF

# Lets store secret envs inside k8s secret

cat <<EOF > very-secure-env.yml
apiVersion: v1
data:
  PASSWORD: c3VwZXJzZWNyZXQ=
kind: Secret
metadata:
  name: very-secure-env

EOF

# Lets create pod and mount both the secret to pod

cat <<EOF > pod.yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: test
  name: test
spec:
  volumes:
    - name: secret-volume
      secret:
        secretName: very-secure-files
  containers:
  - image: nginx
    name: test
    ports:
    - containerPort: 80
    env:
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: very-secure-env
          key: PASSWORD
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
EOF

```


# Now lets hack that secret

## 1) Method: SSH into container - if you have access of K8S API
```bash

k exec -it test -- bash

# GET THE ENV
env | grep -i "password"

# GET THE MOUNT FILE
mount | grep -i "secret"

# GET THE PATH TO FILEs
cd /etc/secret-volume

# LIST THE FILES
ls -a

# READ THE FILE
cat .secretfile

# READ THE FILE
cat myconfigfile
```

## 2) Method: SSH into worker node - if you have access of node
```bash
# Find out where the pod is running
k get pods -o wide

# SSH into that node
ssh worker-1

# Get SUDO Privilage
sudo -i

# Find that running container
# FIRST WAY
crictl ps | grep -i "test"      # get it if you know the name of container

# SECOND WAY
crictl pods --name test -v      # get it if you know the name of pod - you will get pod id from hrer
crictl ps | grep -i "<POD_ID>"
crictl ps -p  "<POD_ID>"


# Lets inspect container - get env
crictl inspect 0af252064f928 | grep -i "env" -A10

# Lets inspect container - get mount path
crictl inspect 0af252064f928 | grep -i "/etc/secret-volume" -A10
crictl inspect 0af252064f928 | grep -i "mount" -A10

# Get the pid
crictl inspect 0af252064f928 | grep -i "pid"

# Read secret
ls -a /proc/22938/root/etc/secret-volume
ls -a /proc/22938/root/etc/secret-volume/.secretfile
ls -a /proc/22938/root/etc/secret-volume/myconfigfile

```


## 3) Method: Access ETCD - if you have access to ETCD
```bash
sudo ETCDCTL_API=3 etcdctl \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key  \
   get /registry/secrets/default/very-secure-files

sudo ETCDCTL_API=3 etcdctl \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key  \
   get /registry/secrets/default/very-secure-env

```


# Encrypton at REST
- No-body should be able to read secret from etcd
- So we want to store secrets, configmap and other things in the encrypted format

```bash

# Create directory
sudo mkdir -p /etc/kubernetes/enc/

# write config file
# this file has configuration how to encrypt etc...

cat <<EOF | sudo tee /etc/kubernetes/enc/enc.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
      - configmaps
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: c2VjcmV0IGlzIHNlY3VyZQ==
      - identity: {} # REMOVE THIS LINE
EOF

# You can edit if you want to
sudo vim /etc/kubernetes/enc/enc.yaml

# Now lets give this file to the Kube-api server
# Open kube-apiserver in the editor mode
# and add below changes
sudo vim /etc/kubernetes/manifests/kube-apiserver.yaml


    - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml  # add this line

    volumeMounts:
    ...
    - name: enc                           # add this line
      mountPath: /etc/kubernetes/enc      # add this line
      readOnly: true                      # add this line
    ...
  volumes:
  ...
  - name: enc                             # add this line
    hostPath:                             # add this line
      path: /etc/kubernetes/enc           # add this line
      type: DirectoryOrCreate             # add this line
```

For more details checkout:
  - https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/



## Scenario - 1
- You have many secrets and all of them are Not encrypted
- you want to encrypt all of them

<!-- Step 1: Belos should be your configurations -->
```yml
cat <<EOF | sudo tee /etc/kubernetes/enc/enc.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: c2VjcmV0IGlzIHNlY3VyZQ==
      - identity: {} # REMOVE THIS LINE
```

<!-- Step 2: Hit below command  -->
```bash
k get secret -A -o yaml | k replace -f -
```

## Scenario - 2
- you have many secrets and all of the are Encrypted
- you want to decrypt all of them

<!-- Step 1: Belos should be your configurations -->
```yml
cat <<EOF | sudo tee /etc/kubernetes/enc/enc.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - identity: {} # REMOVE THIS LINE
      - aescbc:
          keys:
            - name: key1
              secret: c2VjcmV0IGlzIHNlY3VyZQ==
```

<!-- Step 2: Hit below command  -->
```bash
k get secret -A -o yaml | k replace -f -
```