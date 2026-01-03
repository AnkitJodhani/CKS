## Security Context
<!-- Goto:Doc------------ Search: SecurityContext ---->
<!-- Click:FirstLink----- Grab: manifest          ---->

### BASIC
- Run container as non root user
- Since fsGroup field is specified, all processes of the container are also part of the supplementary group ID 2000. The owner for volume `/data/demo` and any files created in that volume will be Group ID 2000


```yml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
    supplementalGroups: [4000]
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-demo
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: false
```

- SSH into the container & perform some test
```bash
k exec -it security-context-demo -- sh

ps

cd /data

# The output shows that the /data/demo directory has group ID 2000, which is the value of fsGroup.
ls -l

cd demo
echo hello > testfile

# The output shows that testfile has group ID 2000, which is the value of fsGroup.
ls -l


cd /tmp
echo hello > testfile

# The output shows that testfile has group ID 3000, coz that is created by user 1000
ls -l

# uid=1000 gid=3000 groups=2000,3000,4000
id

```

### BASIC + 1
- you can also pass securityContext at container level
```yml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
    supplementalGroups: [4000]
  containers:
  - name: sec-ctx-demo-1
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
  - name: sec-ctx-demo-2
    image: nginx
    securityContext:
      runAsNonRoot: false # if this is true then Non-ROOT USER should be mentiond in docker imgae while building or At POD LEVEL
      runAsUser: 0 # Coz nginx needs root permission to server on port 80
      runAsGroup: 4000
```


## Run container in Privilaged mode
- in k8s bydefault, container runs DOES NOT run in previlaged mode
- Privileged: means that container user 0 (root) is directly mapped to host user 0 (root)

```yml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  containers:
  - name: sec-ctx-demo-1
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
```
- Lets Test it
```bash
# we are runing container as ROOT user
id # uid=0(root) gid=0(root) groups=0(root),10(wheel)

# Can't change the HOSTNAME even we are
sysctl kernel.hostname=attacker # sysctl: error: 'kernet.hostname' is an unknown key
```

- Let see how to we can run container in PRIVILEGED mode
```yml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  containers:
  - name: sec-ctx-demo-1
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    securityContext:
      privileged: true
```

- Lets Test it
```bash
# we are runing container as ROOT user
id # uid=0(root) gid=0(root) groups=0(root),10(wheel)

# Now we should be able to do it
sysctl kernel.hostname=attacker  # kernel.hostname = attacker
```

## Disable PrivilegeEscalation
- Allow PrivilegeEscalation controls weather a process can gain more privilege than its parent process
- By default, k8s allow privilege escalation

```yml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  containers:
  - name: sec-ctx-demo-1
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    securityContext:
      allowPrivilegeEscalation: true
```
- Verify
```bash
cat /proc/1/status
# if `NoNewPrivs=0` means PrivilegeEscalation Allowd
```

- set `allowPrivilegeEscalation` to `false` and verify again
```yml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  containers:
  - name: sec-ctx-demo-1
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    securityContext:
      allowPrivilegeEscalation: false
```
- Verify
```bash
cat /proc/1/status
# if `NoNewPrivs=1` means PrivilegeEscalation blocked
```



## Pod Security Policies

- Note: PodSecurityPolicy was deprecated in Kubernetes v1.21, and removed from Kubernetes in v1.25.