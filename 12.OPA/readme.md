## OPS & GateKeeper
- Allow us to implement very customize policies
- General perpose policy agent
- Not specific to Kubernetes
- Easy to impement in REGO language
- Work with JSON/YAML



Install gatekeeper
```bash

kubectl create clusterrolebinding cluster-admin-binding \
    --clusterrole cluster-admin \
    --user kubernetes-admin

helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts

helm install gatekeeper/gatekeeper --name-template=gatekeeper --namespace gatekeeper-system --create-namespace

```

- Link has many examples
https://github.com/open-policy-agent/gatekeeper/tree/master/demo/basic

## Allow if resource have Lable 'gatekeeper'
```yml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        # Schema for the `parameters` field
        openAPIV3Schema:
          type: object
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels

        violation[{"msg": msg, "details": {"missing_labels": missing}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("you must provide labels: %v", [missing])
        }
```
- apply above file, this file will create crds


```yml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: ns-must-have-gk
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
  parameters:
    labels: ["gatekeeper"]

```
- apply above file, this file wil create actual resource


- Lets put above things to TEST
```bash

k create ns temp
# Error from server (Forbidden): admission webhook "validation.gatekeeper.sh" denied the request: [ns-must-have-gk] you must provide labels: {"gatekeeper"}

```

- Create NS with namespace
```yml
apiVersion: v1
kind: Namespace
metadata:
  labels:
    gatekeeper: ankit
  name: temp
```


## Allow resouce to have atleast 2 replicas

```yml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredreplicas
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredReplicas
      validation:
        # Schema for the `parameters` field
        openAPIV3Schema:
          type: object
          properties:
            min:
              type: integer

  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredreplicas

        violation[{"msg": msg, "details": {"missing_replicas": missing}}] {
          provided := input.review.object.spec.replicas
          required := input.parameters.min
          missing := required - provided
          provided < required
          msg := sprintf("you must provide atleast 2 replica and you have  %v replicas ", [missing])
        }
```

- apply above file, this file will create crds


```yml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredReplicas
metadata:
  name: rs-must-have-2-rep
spec:
  match:
    kinds:
      - apiGroups: ["apps"]
        kinds: ["Deployment"]
  parameters:
    min: 2

```
- apply above file, this file wil create actual resource


- Create deployment 1 and 3
```bash
k create deployment nginx-d --image=nginx --port=80
# error: failed to create deployment: admission webhook "validation.gatekeeper.sh" denied the request: [rs-must-have-2-rep] you must provide atleast 2 replicas and currently you have: 1 replicas


k create deployment nginx-d --image=nginx --port=80 --replicas=2

```