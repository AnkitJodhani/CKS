
## Node Metadata Protection
- All the cloud provider has one server called `Metadata server`
- Metadata server has many information about our Virtual Machines
- It also has many **Sensitive** information
- All the VMs can access the metadata server
- Any user or process running inside VM can query and access Metadata server
- so its our responsitbitly to protect Metadata server from malicious user or process

## For AWS
- Instance metadata is data about your instance that you can use to configure or manage the running instance
- Instance metadata properties are divided into categories, for example, host name, events, and security groups
    - Fore more information checkout this: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html
- Use the Instance Metadata Service to access instance metadata
    - Fore more information checkout this: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html

## Practical
- SSH into the EC2 or VM
    ```bash
    TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`

    curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/


    curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id
    ```
- Lets create POD & ssh into it
    ```bash
    k run test --image=nginx --port=80

    k exec -it test -- bash

    # Execute below command inside POD
    curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id

    ```
- As you have noticed you can access the Metada server or user IMDS from Instance as well as POD
- Any Malicious POD can user IMDS and steal some sensitive information


## Lets Protect IMDS or Metadata from group of pods
- Create ns
    ```bash
    k create ns imds
    ```
- Create deployment `prod` (Good actor or Good pod)
    ```bash
    k create deployment prod --image=nginx --port=80 --replicas=2 -n imds
    ```
- Create deployment `test` (Bas actor or Malicious pod)
    ```bash
    k create deployment test --image=nginx --port=80 --replicas=2 -n imds
    ```
- SSH into all the pods and try accessing IMDS or Metadata server, you should be able to acesss it
- Now we want to Restrict access of IMDS or Metadata server to only `prod` deployment so that other pods (i.e test-xxxx) can't access the IMDS or metadata server
- We can utilize network policy to restrict the access

## Network Policy - Restrict access of IMDS
```yml
# ⛔ NO ONE can access the IMDS or Metadata server
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-imds
  namespace: imds
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 169.254.169.254/32

---
# ✅  Allow prod deployment to access the IMDS as well as Everything else
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-imds
  namespace: imds
spec:
  podSelector:
    matchLabels:
      app: prod
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 169.254.169.254/32
```

## 🔥🔥 Note:
- there was problem with cilium network plugin while using ipBlock in the network policy
- when we use ipBlock in netpol while using cilium, IT JUST BLOCK EVERYTHING even if you have specified 0.0.0.0/0
- above netpol was not behaving as expected with cilium
- so i switch from cilium to weave works
- now, above netpol is behaving as expected with weave works CNI


