# Verify Platform Binaries
- When we install any binary or softwere in our system, we
  should check that its NOT Currupted and Not Compromised
- From Softwere or Binary you can generate fingerprint (Hash or string), Its one way process
- Softwere --------- ONE-WAY-PROCESS---------> Fingerprint
- from fingerprint you can generate softwere, ofcoures
- All the softwere maker keep softwere download link & its fingerprint(hash) side by side
- So after downloding softwere you can check the fingerprint to veriy Is it currupted or not?
- To Check the binary is not currputed we use sha512sum algorithms

Note: Goto:------> Kubernetes offcial Githu Repo --------> Click: Tags --------> Pick your k8s version
          -------> Click: See the CHANGELOG  ----------> Scroll Down -------> will find client or server binaries
          -------> All the Binary download along with its Fingerprint (HASH) -----> Download Binary in your system

## Practical
- Follow above process and download any server binary

```bash

# Download Binary
wget wget https://dl.k8s.io/v1.30.9/kubernetes-server-linux-amd64.tar.gz


# Generate HASH from locally from binary
sha512sum kubernetes-server-linux-amd64.tar.gz


# put HASH into file for processing
sha512sum kubernetes-server-linux-amd64.tar.gz | awk '{print $1}' > compare.txt

# COPY actual HASH (Fingerprint) from the Downloaded SITE and put in the same file
echo "a80aa58418d6f25e4fa82f54762033a6634b310446226dda4852924cf19766268e97ead6f1a93ad72d89d3a519073658b12be0709b3277d4f0e470e26f5e606d" >> compare.txt

# Output only unique HASH or STRING or Fingerprint,
cat compare.txt | uniq

# IF Binary is NOT Currpted ------->  YOU WILL SEE ONLY ONE STRING or HASH or FINGERPRINT
# IF Binary is  Currpted    ------->  YOU WILL SEE TWO  STRING or HASH or FINGERPRINT, coz both are different


```


## Example - Compare the Hash of Running kube-apiserver with Actual from the offcial repository

1) Check the version of running kuber-apiserver
```bash
kgp -n kube-system kube-apiserver-master -o yaml | grep -i "image"
```

2) Check the system architecture
```bash
arch
uname -m

# x86_64 --------------------> AMD 64-bit processor
# aarch64  or arm64 ---------> ARM64 architecture
```

3) Download official Server binaries from official github repository
Note: Goto:------> Kubernetes offcial Githu Repo --------> Click: Tags --------> 🔥Pick your k8s version
          -------> Click: See the CHANGELOG  ----------> Scroll Down -------> you will find Server binaries
          -------> 🔥 Download Binary which is compatible your system
```bash

# Down at the official server binary zip
wget https://dl.k8s.io/v1.30.9/kubernetes-server-linux-amd64.tar.gz

# Unzip the zip file
tar xzvf kubernetes-server-linux-amd64.tar.gz

# Get into the below location
cd kubernetes/server/bin/

# list out all the binary - kube-apiserver, kube-proxy etc...
ls

# Generate sum of kube-apiserver and store into compare file
sha512sum kube-apiserver | awk '{print $1}' > compare


# Now we need to find out the binary Running kube-apiserver container
# We CAN NOT ssh into the kube-api server pod, coz it does not contain shell
# So we need to access binary file from the host(master node) file system
# 🤩 See below How can we access the file system of running container(kube-apiserver)


# List of all the container of master node
crictl ps

# All container running on the HOST, so each container will have its PROCESS running in the host
# So grab the PROCESS ID of process
ps aux | grep -i "kube-apiserver"

# List of the root file system of the container
sudo ls /proc/9032/root/


# Findout where the kuber-apiserver binary is located inside container
sudo find /proc/9032/root/ | grep -i "kube-apiserver"

# Generate HASH of that binary and store into the compare file
sudo sha512sum /proc/9032/root/usr/local/bin/kube-apiserver | awk '{print $1}' >> compare

# compare both HASHshsss
cat compare | uniq
```


