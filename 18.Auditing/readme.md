## Auditing in K8s
- Auditing allows cluster administrators to answer the following questions:
    - what happened?
    - when did it happen?
    - who initiated it?
    - on what did it happen?
    - where was it observed?
    - from where was it initiated?
    - to where was it going?


## Stages - Each request can be recorded with an associated stage
- `RequestReceived`
- `ResponseStarted`
- `ResponseComplete`
- `Panic`


## Level - How deep(informative) you want the log
- `None`
- `Metadata`
- `Request`
- `RequestResponse`

## Practical

- Create directory
```bash
sudo mkdir -p /etc/kubernetes/audit

cd /etc/kubernetes/audit
```

- create policy at location `/etc/kubernetes/audit/audit-policy.yaml`
```bash
apiVersion: audit.k8s.io/v1 # This is required.
kind: Policy
rules:
  - level: Metadata
```

- configure that in the `kube-apiserver.yaml`
```bash
sudo vim /etc/kubernetes/manifests/kube-apiserver.yaml

    - --audit-policy-file=/etc/kubernetes/audit/audit-policy.yaml
    - --audit-log-path=/etc/kubernetes/audit/audit.json
    - --audit-log-maxage=2
    - --audit-log-maxbackup=5
    - --audit-log-maxsize=100

    - mountPath: /etc/kubernetes/audit
      name: audit

  - hostPath:
      path: /etc/kubernetes/audit
      type: DirectoryOrCreate
    name: audit
```


## Create audit policy for below things
Nothing from stage `RequestReceived`
Nothing from "get", "watch", "list"
From Secrets only metadata level
Everything else `RequestResponse` level


```yml
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - "RequestReceived"
rules:
  - level: None
    verbs: ["get","list","watch"]
  - level: Metadata
    resources:
    - group: ""
      resources: [secretes"]
  - level: RequestResponse

```