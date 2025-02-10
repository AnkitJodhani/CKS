## mTLS
- we should encrypt POD to POD communication
- Istio & LinkerD help us in achiving that


```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: test
  name: test
spec:
  containers:
  - image: bash
    name: talker
    command:
    - sh
    - -c
    - ping google.com
  - image: ubuntu
    name: proxy
    command:
    - sh
    - -c
    - 'apt-get update && apt-get install iptables -y && iptables -L && sleep 1d'
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]

```

- See the OUTPUT of the IPTABLES
- We can use IPTABLES to implement new rules
```bash
k logs test -c proxy
```

