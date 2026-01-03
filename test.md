

---
###  What is Vertical Pod AutoScaler?
---

**Purpose**: Automatically recommends or updates CPU/memory resource requests & limits for Pods, adjusting “vertically” instead of adding replicas

**Requirement**: A component exposing the resource.metrics.k8s.io API (commonly the Metrics Server) must be running so VPA can fetch per-container CPU and memory usage data

**Requirement**: VPA needs either Metrics Server or another implementation of the k8s “resource metrics” API `resource.metrics.k8s.io` (e.g., Prometheus Adapter)

**Needs VPA Controllers**: VPA Controllers need to be installed

**Modes**:
 - **Off**: No changes to pods; just stores/suggests recommendations and can be inspected in the VPA object
 - **Initial**: Sets recommended resources only at pod creation. Does not evict or update existing pods
 - **Recreate**: Sets recommended resources when new pods are created and evicts existing pods when the requested resources differ significantly from the new recommendation (respecting the PDBs, if defined)
 - **Auto**(Deprecated): Equivalent to `Recreate` - will be removed in a future API version
 - **InPlaceOrRecreate**(alpha feature): Assigns resource requests on pod creation as well as updates them on existing pods by leveraging Kubernetes `in-place update` capability



```yaml
# VPA to auto-adjust resources for 'web-app' pods
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: web-app-vpa               # VPA resource name
spec:
  targetRef:                      # Target to apply VPA on
    apiVersion: apps/v1           # Target API version
    kind: Deployment              # Kind: Deployment
    name: web-app                 # Name of the deployment
  updatePolicy:
    updateMode: "Recreate"        # Use explicit mode instead of deprecated "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: "*"          # Applies to all containers
      minAllowed:                 # Minimum resource limits
        cpu: "100m"               # At least 100 millicores
                                  # (0.1 vCPU)
        memory: "128Mi"           # At least 128Mi memory
      maxAllowed:                 # Maximum resource limits
        cpu: "1"                  # No more than 1 CPU
        memory: "1Gi"             # No more than 1Gi memory
```