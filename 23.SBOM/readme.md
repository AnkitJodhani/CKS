# SBOM - Software Bill of Materials
- We use lot of OpenSource software in our software product
- There are many opensource libraries & packages present in out s/w
- There are chances that attacker can attack such OpenSource s/w
- So its IMP to track the entire software link & list of lib + packages

## Question - needs to be answered
- Who is building?
- What's being build?
- Where's the software being build?

# BOM
```bash

bom --help

# Generate a SPDX-Json SBOM of image & store it somewhere
bom generate --image <NAME_OF_IMAGE> --format json --output <LOCATION>

```


# Trivy
```bash

trivy --help

trivy image --help

# Generate a CycloneDX SBOM of image & store somewhere
trivy image <NAME_OF_IMAGE> --format <SPECIFY_FORMAT> --output <LOCATION>


# With Trivy we can also scan SBOM documents instead of images directly
trivy sbom <EXISTING_SBOM_FILE> --format json --output <FINAL_RESULT_FILE_LOCATION>

# For exmple
trivy sbom --format json --output /opt/course/18/sbom_check_result.json /opt/course/18/sbom_check.json


```




