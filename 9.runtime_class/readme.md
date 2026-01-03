## Container Runtime Sandboxes
- we will use Gvisor as Container Runtime Sandboxes
- Gvisor is USER-SPACE Kernel
- Gvisor will limit container to make SYSTEMCALL
- use below script as run it on `worker-1`

```bash
sudo apt-get update && \
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg


curl -fsSL https://gvisor.dev/archive.key | sudo gpg --dearmor -o /usr/share/keyrings/gvisor-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/gvisor-archive-keyring.gpg] https://storage.googleapis.com/gvisor/releases release main" | sudo tee /etc/apt/sources.list.d/gvisor.list > /dev/null



sudo apt-get update && sudo apt-get install -y runsc


cat <<EOF | sudo tee /etc/containerd/config.toml
version = 2
[plugins."io.containerd.runtime.v1.linux"]
  shim_debug = true
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  runtime_type = "io.containerd.runc.v2"
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runsc]
  runtime_type = "io.containerd.runsc.v1"
EOF

sudo systemctl restart containerd
sudo systemctl restart kubelet

```

## SSH into `worker-1`
- lets try to do the systemcall
```bash
uname -r # will do the SYSTEMCALL - Note the output
```

## Schedule pod on `worker-1`
- schedule pod on `worker-1`
- SSH into the POD and hit 'uname -r'
- See are you getting same output as worker-1?
- if yes, it means you container or POD can directly perform SYSTEMCALL
- not good idea

```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: test-1
  name: test-1
spec:
  nodeName: worker-1
  containers:
  - image: nginx
    name: test-1
    ports:
    - containerPort: 80
```
```bash
k exec -it test-1 -- bash
uname -r
dmesg
# Output: dmesg: read kernel buffer failed: Operation not permitted
```


## Lets add Layer between Container and Kernet
- CONTAINER -------> Gvisor (runsc) ----------> Kernel
- Gvisor will limit container to perform SYSTEMCALL
- We already installd Gvisor(runsc) on `worker-1`
<!----Goto: Doc ------ Search: runtimeclass--------Clike: first link
      Grab the manifest files -->


- `RuntimeClass` specify which runtime do you want to utilize
```yml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: myclass
handler: runsc
```

- Instruct POD to use specified runtimeclass using field `runtimeClassName`
```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: test-2
  name: test-2
spec:
  runtimeClassName: myclass
  nodeName: worker-1
  containers:
  - image: nginx
    name: test-2
    ports:
    - containerPort: 80
```

- Once the pod comes in running state, SSH into POD
- execute `uname -r`
- you will get different output then you've got when hitting into `worker-1` or `test-1` pod

```bash
k exec -it test-2 -- bash
uname -r
dmesg

```
