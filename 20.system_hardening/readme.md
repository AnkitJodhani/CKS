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
kaf pod.yml

# Verify
# You can verify that the container is actually running with that profile by checking /proc/1/attr/current:
kubectl exec hello-apparmor -- cat /proc/1/attr/current

# Finally, you can see what happens if you violate the profile by writing to a file:
kubectl exec hello-apparmor -- touch /tmp/test
```