## ImagePolicyWebhook
- The ImagePolicyWebhook admission controller allows a backend webhook to make admission decisions


---> Api Req ------> APIserver ----> Allow/Deny
                        |
                        |
                        |
                        +
                      ⬇️⬇️
               AdmissionControllers
                        |
                        |
                        |
                        +
                      ⬇️⬇️
               ImagePolicyWebHook
                        |
                        |
                        |
                        +
                      ⬇️⬇️
                ExternalServices

- ExternalServices will receive object of kind: `ImageReview`

- https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook

```bash
# add below line
sudo vim /etc/kubernetes/manifests/kube-apiserver.yaml

    - --enable-admission-plugins=NodeRestriction,ImagePolicyWebhook

```
