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

# 👇  Step 1: Create directory
sudo mkdir -p /etc/kubernetes/ipw/

# 👇 Step 2: Create config file for admission controller
cat <<EOF | sudo tee /etc/kubernetes/ipw/admissionconfig.yml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
  - name: ImagePolicyWebhook
    path: /etc/kubernetes/ipw/ipwconfig.yml
EOF

# 👇 Step 3: Create config file for ImagePolicyWebhook
cat <<EOF | sudo tee /etc/kubernetes/ipw/ipwconfig.yml
imagePolicy:
  kubeConfigFile: /etc/kubernetes/ipw/backendconfig
  # time in s to cache approval
  allowTTL: 50
  # time in s to cache denial
  denyTTL: 50
  # time in ms to wait between retries
  retryBackoff: 500
  # determines behavior if the webhook backend fails
  defaultAllow: false
EOF

# 👇 Step 4: Create backendconfig file for backend service so that api server can see the endpoint and certificate for communication
cat <<EOF | sudo tee /etc/kubernetes/ipw/backendconfig
apiVersion: v1
kind: Config
clusters:
  - name: name-of-remote-imagepolicy-service
    cluster:
      certificate-authority: /path/to/ca.pem
      server: https://images.example.com/policy # URL of remote service to query. Must use 'https'.

contexts:
- context:
    cluster: name-of-remote-imagepolicy-service
    user: name-of-api-server
  name: name-of-api-server
current-context: name-of-api-server

# users refers to the API server's webhook configuration.
users:
  - name: name-of-api-server
    user:
      client-certificate: /path/to/cert.pem # cert for the webhook admission controller to use
      client-key: /path/to/key.pem          # key matching the cert
EOF


# 👇  Step 5: EDIT API Server configuratin to let it know about ImagePolicyWebhook
sudo vim /etc/kubernetes/manifests/kube-apiserver.yaml

    - --enable-admission-plugins=NodeRestriction,ImagePolicyWebhook

    - --admission-control-config-file=/etc/kubernetes/ipw/admissionconfig.yml

    volumeMounts:
    - mountPath: /etc/kubernetes/ipw
      name: ipw
      readOnly: true
  volumes:
  - hostPath:
      path: /etc/kubernetes/ipw
      type: DirectoryOrCreate
    name: ipw
```
