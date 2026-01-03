## System Hardening tools
- AppArmor
- Seccomp


## AppArmor
- 3 types of profile
  - **Unconfined** - Process can escape
  - **Complain** - Process can escape but it will be logged
  - **Enforce** - Process can not escape


- Install AppArmor
```bash
sudo apt install apparmor-profiles

sudo apt install apparmor-utils -y
```

```bash

# show all the profiles
aa-status

# generate a new profile
aa-genprof

# put profile in complain mode
aa-complain

# put profile in enforce mode
aa-enforce

# update the profile if app produced some more usages logs (syslog)
aa-logprof


# create profile for curl & pfress 'F' to block eveything
aa-genprof curl

# All the profiles are located here
ls /etc/apparmor.d/

# Test - you won't get any response - coz AppArmor BLOCKED everything
curl https://www.google.com

# How to enable functionality based on the uses

# Read syslogs - AppArmor will read the logs and generate appropriate access
cat /var/log/syslog | grep -i "curl"

# It will ask you some question and then give appropriate access
aa-logprof

# Test it again
curl https://www.google.com


```



## AppArmor with Pods
- Install AppArmor on `worker-1` node
- Configure the profile so that pods can utilize it
```bash
cat <<EOF > /etc/apparmor.d/k8s-apparmor-example-deny-write

#include <tunables/global>

profile k8s-apparmor-example-deny-write flags=(attach_disconnected) {
  #include <abstractions/base>

  file,

  # Deny all file writes.
  deny /** w,
}
EOF


sudo apparmor_parser /etc/apparmor.d/k8s-apparmor-example-deny-write
```

- Lets create pod that will utilize above create profile
```bash
# Find pod.yml in the same directory
kaf pod-apparmor.yml

# Verify
# You can verify that the container is actually running with that profile by checking /proc/1/attr/current:
kubectl exec hello-apparmor -- cat /proc/1/attr/current

# Finally, you can see what happens if you violate the profile by writing to a file:
kubectl exec hello-apparmor -- touch /tmp/test
```


## Seccomp
- Secure computing mode
- Restrict execution of syscall

- Practical info: https://kubernetes.io/docs/tutorials/security/seccomp/

```bash
# Note: SSH into worker-1

sudo mkdir -p /var/lib/kubelet/seccomp/profiles

cd  /var/lib/kubelet/seccomp/profiles

curl -L -o profiles/audit.json https://k8s.io/examples/pods/security/seccomp/profiles/audit.json

curl -L -o profiles/violation.json https://k8s.io/examples/pods/security/seccomp/profiles/violation.json

curl -L -o profiles/fine-grained.json https://k8s.io/examples/pods/security/seccomp/profiles/fine-grained.json

ls profiles


# Lets write pod manifest
cat <<EOF > pod-seccomp.yml
apiVersion: v1
kind: Pod
metadata:
  name: audit-pod
  labels:
    app: audit-pod
spec:
  nodeName: worker-1
  securityContext:
    seccompProfile:
      type: Localhost
      # localhostProfile: profiles/audit.json          # Print to info
      # localhostProfile: profiles/violation.json      # Print violation info
      localhostProfile: profiles/fine-grained.json     # It gives necessary permisson
  containers:
  - name: test-container
    image: hashicorp/http-echo:1.0
    args:
    - "-text=just made some syscalls!"
    securityContext:
      allowPrivilegeEscalation: false
EOF


kaf pod-seccomp.yml


kubectl expose pod audit-pod --type NodePort --port 5678

# Again, Lets SSH into Worker-1
curl http://localhost:NODE_PORT

# Lets checkout the logs which are printed by the seccomp
cat /var/log/syslog
cat /var/log/syslog | grep -i "http"


```