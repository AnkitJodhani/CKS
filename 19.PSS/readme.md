# Pod Security Standers
- Define 3 different policies to broadly cover the security spectrum
  - `Privileged`
  - `Baseline`
  - `Restricted`

# Pod Security Admission
- Built-in Pod Security admission controller to enforce the Pod Security Standards
- Pod security restrictions are applied at the namespace level when pods are created



```bash
cat <<EOF > ns.yml
apiVersion: v1
kind: Namespace
metadata:
  name: test
  labels:
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/enforce-version: v1.32
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/warn-version: v1.32
EOF


cat <<EOF > deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-d
  name: nginx-d
  namespace: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-d
  template:
    metadata:
      labels:
        app: nginx-d
    spec:
      hostPID: true  # 🚫 Not allowd in th baseline policy
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
        securityContext:
          privileged: true  # 🚫 Not allowd in th baseline policy
          allowPrivilegeEscalation: true

EOF

# Lets apply the files
kaf ns.yml

# You should receive the warning because of "restricted" policy
kaf depoyment.yml

# Lets see if there is any pod got created?
# you won't see any pod
kgp -n test

# Let see the deployment
kgd -n test

# Lets see the replica set
# you notice that rs does not have any ready pod
k get rs -n test

# Lets describe the events of the rs
# you wil the  error about "violates PodSecurity baseline:v1.32"
kdesc rs -n test nginx-d-5fbc7fc5f8


```





