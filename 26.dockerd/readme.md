# Expose dockerd on port 2375
Why a TCP Port?
- Remote Access: Exposing the daemon on a TCP port (e.g., 2375) allows remote clients to connect.
- Default Ports:
    - `2375`: Commonly used for unencrypted (plain HTTP) connections.
    - `2376`: Used when securing the connection with TLS (recommended for production).

## First verify - Is it listning to any PORT?
```bash
# First check is dockerd exposed on any port?
# try looking for "-H tcp://0.0.0.0:PORT"
# try looking for "-H tcp://0.0.0.0:2375"
sudo systemctl cat docker.service

# You are not able to see the line "-H tcp://0.0.0.0:2375"
# it means docker daemon is not listning on ary port

# Note: you might see this line"-H fd:// --containerd=/run/containerd/containerd.sock"
# it means configured to listen on the Unix socket
```

## Lets make it listen on PORT 2375
```bash
# Get the path of systemfile - you will get path in the first line itself
sudo systemctl cat docker.service

# Open system file in the vim editor
sudo vim /usr/lib/systemd/system/docker.service
# or another way
# export EDITOR=vim
# systemctl edit docker.service

# Add following attribute in line `ExecStart` & save it
# "-H tcp://0.0.0.0:2375"
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375 --containerd=/run/containerd/containerd.sock


# Lets reload and restart service
sudo systemctl daemon-reload
sudo systemctl restart docker.service

```

## Now its time to verify - Is Dockerd listning on any PORT?
```bash
sudo apt-get install net-tools -y

docker -H tcp://<your-host-ip>:2375 info
docker -H tcp://127.0.0.1:2375 info

curl http://localhost:2375/version

```