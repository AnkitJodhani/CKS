# Detect runtime activity
- Install falco
- Goto Doc --------> Click: Reference -------> Click: Rules Examples
  Copy Rules ------> Click:Default Macros ---> Copy syscall
  Click: Supported Events -------------------> Click: Fields for Conditions and Outputs


```bash

# Falco files are located at
cd /etc/falco/

# What are the files present here
-rw-r--r-- 1 root root  1096 Mar  9 15:32 falco_rules.local.yaml    # ADD YOUR CUSTOM RULE HERE
-rw-r--r-- 1 root root 63723 Mar  9 15:18 falco_rules.yaml          # DEFAULT RULES ARE HERE
drwxr-xr-x 2 root root  4096 Jan 28 09:34 config.d
drwxr-xr-x 2 root root  4096 Jan 28 09:34 rules.d                   # YOU CAN ALSO ADD YOUR RULES HERE
-rw-r--r-- 1 root root 58628 Jan 28 09:06 falco.yaml                # FALCO DEFAULT CONFIG

# To see list of files are applied - and the order of files
cat falco.yaml | grep -i "rules_files" -A5

# To see where the output is going
cat falco.yaml | grep -i "file_output" -A5

# if you made any mistake while writing the rules
# hit below command to debug

```

# Lets do the Practical
## Step 1: Create POD that access the /dev/mem file
## Step 2: Create Falco rule detect that changes

```bash
cat <<EOF | tee pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: devmem-test
spec:
  nodeName: worker-1
  containers:
  - name: devmem-access
    image: busybox
    command: ["sh", "-c", "while true; do dd if=/dev/mem bs=1 count=10 2>/dev/null || echo 'Access failed'; sleep 1; done"]
    securityContext:
      privileged: true
    volumeMounts:
    - name: devmem
      mountPath: /dev/mem
  volumes:
  - name: devmem
    hostPath:
      path: /dev/mem
      type: CharDevice
EOF



cat <<EOF | sudo tee /etc/falco/rules.d/custom_rules_1.yaml
- rule: Will detech if file /dev/mem is opend for anything
  desc: ok
  condition: evt.type in (open,openat,read,write) and fd.name = "/dev/mem"
  enabled: true
  output: Alert: HERE IS THE CULPRIT container=%container.id pod=%k8s.pod.name user=%user.uid username=%user.name process=%proc.cmdline
  priority: CRITICAL
EOF

cat <<EOF | sudo tee /etc/falco/rules.d/custom_rules_2.yaml
- rule: "Monitor activity in /dev and /tmp directories"
  desc: "Alert on any file activity in /dev or /tmp directories"
  condition: fd.name startswith "/dev/" or fd.name startswith "/tmp/"
  output: "Alert: File activity in sensitive directory detected: %fd.name container=%container.id pod=%k8s.pod.name process: %proc.cmdline (user=%user.name)"
  priority: WARNING
  tags: [filesystem, sensitive, /dev, /tmp]
EOF
```