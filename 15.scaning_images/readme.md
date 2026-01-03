## Scaning images
- Webserver & apps may contain Vulnerabilities

## Tools
- Clair
    - Open source
    - Static analysis of vulnerabilities in app containers
    - provide API
- Trivy
    - Open source
    - simple, easy & fast vulnerabilities scanner


```bash

k run trivy --image=aquasec/trivy --command trivy image python:3.4-alpine

k logs trivy

```

- If you have installed as CLI
```bash
trivy image <NAME_OF_THE_IMAGE>

trivy --severity=HIGH,CRITICAL image <NAME_OF_THE_IMAGE>

```