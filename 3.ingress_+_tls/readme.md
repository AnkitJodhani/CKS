

## Install nginx ingress controller

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update


helm install ingress-nginx ingress-nginx/ingress-nginx

<!-- verify installation -->
kgp


<!-- verify service created by the nginx-ingress controller -->
<!-- Note: you should also see the nodePort service port 80 and 443 -->
kgs

## Lets do the testing of the Nginx ingress controller
1)  Testing with HTTP
curl http://NODE_IP_ADDRESS:HTTP_NODE_PORT/
- expected response could be
  ```
    <html>
    <head><title>404 Not Found</title></head>
    <body>
    <center><h1>404 Not Found</h1></center>
    <hr><center>nginx</center>
    </body>
    </html>
  ```
2) Testing with HTTPS
curl https://NODE_IP_ADDRESS:HTTPS_NODE_PORT/ -v
- expected response could be
  ```
    curl: (60) SSL certificate problem: self-signed certificate
    More details here: https://curl.se/docs/sslcerts.html

    curl failed to verify the legitimacy of the server and therefore could not
    establish a secure connection to it. To learn more about this situation and
    how to fix it, please visit the web page mentioned above.
  ```
curl https://NODE_IP_ADDRESS:HTTPS_NODE_PORT/ -kv
- expected response could be
  ```
    ......ABOVE
    * Server certificate:
    *  subject: O=Acme Co; CN=Kubernetes Ingress Controller Fake Certificate
    *  start date: Jan 18 09:00:44 2025 GMT
    *  expire date: Jan 18 09:00:44 2026 GMT
    *  issuer: O=Acme Co; CN=Kubernetes Ingress Controller Fake Certificate
    *  SSL certificate verify result: self-signed certificate (18), continuing anyway.
    * Using HTTP2, server supports multiplexing
    * Connection state changed (HTTP/2 confirmed)
    .....BELOW
    <html>
    <head><title>404 Not Found</title></head>
    <body>
    <center><h1>404 Not Found</h1></center>
    <hr><center>nginx</center>
    </body>
    </html>
    * TLSv1.2 (IN), TLS header, Supplemental data (23):
  ```
- See https is NOT WORKING here. Its Ingress Controller Fake Certificate as you have notieced
- but in the later part of the seciton will generate our own TLS

Note: if you are getting same reposen like me then your cetificate is Nginx ingress contoller is working


## Lets create 2 deployment
k create deployment go-deployment --image=ankitjodhani/golang:latest --replicas=3 --port=3000


k create deployment python-deployment --image=ankitjodhani/python:latest --replicas=3 --port=3000


## Lets create service that expose those 2 deployment

k expose deployment go-deployment --name go-svc --port=80 --target-port=3000

k expose deployment python-deployment --name python-svc --port=80 --target-port=3000


## Lets create ingress that route traffic to those pods
<!-- ingress.yml -->
<!-- Goto: Kuberntres Doc -> search "ingress" -> Grab the manifest -->
```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /go(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: go-svc
            port:
              number: 80
      - path: /python(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: python-svc
            port:
              number: 80
```

Note:
- above ingress wil route `/go` request to `go-svc` service
- above ingress wil route `/python` request to `go-python` service


## Lets do the Testing - Without HTTPS

curl http://WORKER_NDOE_IP:HTTP_NODE_PORT/go/golang -vv

curl http://WORKER_NDOE_IP:HTTP_NODE_PORT/python/python -vv


## Lets apply the TLS

1) Lets first generate certificate(public key)  and private key
<!-- Goto: Kuberntres Doc -> search "Generate Certificates Manually" -> Grab commands -->

```bash
openssl genrsa -out ca.key 2048

# Enter domain name (i.e ankit.com)
openssl req -x509 -new -nodes -key ca.key -days 10000 -out ca.crt

ls

```
- Now you will have `ca.crt` (certificate or public key) and `ca.key` (private key)



2) Lets create secret that will hold certificate and private key
```bash
k create secret tls my-ingress-secret --cert=ca.crt --key=ca.key
```

3) Lets configure the ingress with TLS
<!-- Goto: Kuberntres Doc -> search "ingress" -> Goto: TLS -> Grab manifest -->

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  tls:
  - hosts:
      - ankit.com
    secretName: my-ingress-secret
  ingressClassName: nginx
  rules:
  - host: ankit.com
    http:
      paths:
      - path: /go(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: go-svc
            port:
              number: 80
      - path: /python(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: python-svc
            port:
              number: 80
```
- now lets try to verify Is TLS working or not?


## Lets do the Testing - With HTTPS

curl https://ankit.com:HTTPS_NODE_PORT/go/golang -kv --resolve ankit.com:HTTPS_NODE_PORT:NODE_IP_ADDRESS

- For example( curl https://ankit.com:30976/go/golang -kv --resolve ankit.com:30976:18.207.180.23 )
- Expected output could be
  ```
    ..........ABOVE
    * Server certificate:
    *  subject: C=AU; ST=Some-State; O=Internet Widgits Pty Ltd; CN=ankit.com
    *  start date: Jan 18 09:35:44 2025 GMT
    *  expire date: Jun  5 09:35:44 2052 GMT
    *  issuer: C=AU; ST=Some-State; O=Internet Widgits Pty Ltd; CN=ankit.com
    *  SSL certificate verify result: self-signed certificate (18), continuing anyway.
    * Using HTTP2, server supports multiplexing
    * Connection state changed (HTTP/2 confirmed)
    * Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
    * TLSv1.2 (OUT), TLS header, Supplemental data (23):
    * TLSv1.2 (OUT), TLS header, Supplemental data (23):
    * TLSv1.2 (OUT), TLS header, Supplemental data (23):
    * Using Stream ID: 1 (easy handle 0x560c0e493e90)
    * TLSv1.2 (OUT), TLS header, Supplemental data (23):
    > GET /go/golang HTTP/2
    > Host: ankit.com:30976
    > user-agent: curl/7.81.0
    > accept: */*
    >
    ..........BELOW
  ```
  - as you can see in above output that Now its not fake certificate