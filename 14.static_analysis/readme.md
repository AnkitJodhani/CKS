## Static Analysis
- Look at your source code & text file
- Check aginst rules
- Enforce rules
- Rules can be like
    - POD should never use 'default' SA
    - Always defined Resource req & limit

## Kubesec
- Security risk analysis for Kubernetes resources
- OpenSources
- Opinionated!! Fixed setup of rules

- Run as:
    - Binary
    - Docker container
    - Kubectl Plugin
    - Admisison Controller (kubesec-webhook)

- Install kubesec binary

```bash
wget https://github.com/controlplaneio/kubesec/releases/download/v2.14.2/kubesec_linux_amd64.tar.gz

tar -xzvf kubesec_linux_amd64.tar.gz

sudo mv kubesec /usr/bin/

kubesec

kubesec scan pod.yml

kubesec scan deployment.yml

```

## Kube-linter

- Install kube-linter
```bash
wget https://github.com/stackrox/kube-linter/releases/download/v0.7.1/kube-linter-linux.tar.gz

tar -xzvf kube-linter-linux.tar.gz

sudo mv kube-linter /usr/bin/

kube-linter lint pod.yml

kube-linter lint deployment.yml

```

## Conftest - OPA
- Unit test framework for kubernetes configurations
- Uses Rego language

- install conftest
```bash
LATEST_VERSION=$(wget -O - "https://api.github.com/repos/open-policy-agent/conftest/releases/latest" | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/' | cut -c 2-)
ARCH=$(arch)
SYSTEM=$(uname)
wget "https://github.com/open-policy-agent/conftest/releases/download/v${LATEST_VERSION}/conftest_${LATEST_VERSION}_${SYSTEM}_${ARCH}.tar.gz"
tar xzf conftest_${LATEST_VERSION}_${SYSTEM}_${ARCH}.tar.gz
sudo mv conftest /usr/local/bin

```


#### Use this with Kubernetes
- https://www.conftest.dev/
- https://www.conftest.dev/install/

```bash
mkdir policy

cat <<EOF > policy/deployment.rego
package main

deny[msg] {
  input.kind == "Deployment"
  not input.spec.template.spec.securityContext.runAsNonRoot

  msg := "Containers must not run as root"
}

deny[msg] {
  input.kind == "Deployment"
  not input.spec.selector.matchLabels.app

  msg := "Containers must provide app label for pod selectors"
}
EOF


conftest test deployment.yml

```
#### Use this with Dockerfile
- https://www.conftest.dev/examples/
- https://github.com/open-policy-agent/conftest/tree/master/examples/docker

```bash
cat <<EOF > Dockerfile
FROM openjdk:8-jdk-alpine

VOLUME /tmp

ARG DEPENDENCY=target/dependency

COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY ${DEPENDENCY}/META-INF /app/META-INF
COPY ${DEPENDENCY}/BOOT-INF/classes /app

RUN apk add --no-cache python3 python3-dev build-base && pip3 install awscli==1.18.1

ENTRYPOINT ["java","-cp","app:app/lib/*","hello.Application"]
EOF


cat <<EOF > policy/images.rego
package main

denylist := ["openjdk"]

deny[msg] {
	some i
	input[i].Cmd == "from"
	val := input[i].Value
	contains(val[i], denylist[_])

	msg = sprintf("unallowed image found %s", [val])
}
EOF

cat <<EOF > policy/commands.rego
package commands

denylist := [
	"apk",
	"apt",
	"pip",
	"curl",
	"wget",
]

deny[msg] {
	some i
	input[i].Cmd == "run"
	val := input[i].Value
	contains(val[_], denylist[_])

	msg := sprintf("unallowed commands found %s", [val])
}
EOF

conftest test Dockerfile

```