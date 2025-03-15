## Configure Service Accounts for Pods
- https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

## SA Automounting
- if field `automountServiceAccountToken` specified in the SA and POD then, pod will have more priority (it was the one who work at ground level)


```bash

cat <<EOF | tee sa.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ankit
automountServiceAccountToken: false   # Do NOT Mount automatically
EOF



cat <<EOF | tee pod.yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: test
  name: test
spec:
  automountServiceAccountToken: true   # Mount Automatically, default true
  serviceAccountName: ankit
  containers:
  - image: nginx
    name: test
    ports:
    - containerPort: 80
EOF

```

## Manually create a long-lived API token for a ServiceAccount

```bash

cat <<EOF | tee sa.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ankit
EOF

cat <<EOF | tee secret.yml
apiVersion: v1
kind: Secret
metadata:
  name: ankit-secret
  annotations:
    kubernetes.io/service-account.name: ankit
type: kubernetes.io/service-account-token
EOF

# CA.Crt and TOKEN will be auto Populated inside secret
k get secret ankit-secret -o yaml
```

## Add ImagePullSecrets to a service account
- we don't want to add `imagePullSecrets` field for all the pods
- we want SA to automatically attach that field `imagePullSecrets`

```bash

kubectl create secret docker-registry myregistrykey \
  --docker-server=DOCKER_REGISTRY_SERVER \
  --docker-username=DOCKER_USER \
  --docker-password=DOCKER_PASSWORD \
  --docker-email=DOCKER_EMAIL

cat <<EOF | tee sa.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ankit
imagePullSecrets:
  - name: myregistrykey
EOF

cat <<EOF | tee pod.yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: test
  name: test
spec:
  serviceAccountName: ankit
  containers:
  - image: nginx
    name: test
    ports:
    - containerPort: 80
EOF

```

