---
layout: page
title: "Kubernetes Security"
week_number: "54"
release_date: "2027-04-27"
phase_label: "Phase 11"
phase_name: "Advanced DevOps"
phase_color: "#06b6d4"
permalink: /week-54/
---

> Week 54. Securing Kubernetes clusters.
> 
> Kubernetes is powerful but ships with insecure defaults. Misconfigured clusters are a common attack vector -- leaked kubeconfig files, overprivileged pods, and exposed dashboards have all led to major breaches.
> 
> This week you learn how attackers target Kubernetes and how to harden clusters against them.

---

# Kubernetes Security

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Explain the Kubernetes attack surface and common attack techniques
- Configure RBAC to enforce least privilege
- Apply NetworkPolicies to restrict pod communication
- Set Pod security contexts to reduce container privileges
- Use kube-bench and Trivy to audit a cluster

---

## Why Kubernetes Security Matters

A misconfigured Kubernetes cluster can lead to:

- **Full cluster takeover** - via exposed API server or dashboard
- **Container escape** - privileged container breaks out to the host
- **Lateral movement** - compromised pod pivots to other services
- **Credential theft** - service account tokens stolen from pod filesystem
- **Cryptomining** - attacker deploys workloads on your compute

The Microsoft Kubernetes threat matrix maps 12 attack technique categories. Understanding them helps you know what to defend.

---

## RBAC: Role-Based Access Control

RBAC controls what Kubernetes API calls a user or service account can make.

### Key Objects

- **Role** - grants permissions within a namespace
- **ClusterRole** - grants permissions cluster-wide
- **RoleBinding** - binds a Role to a user/group/service account
- **ClusterRoleBinding** - binds a ClusterRole cluster-wide

### Example: Read-Only Role

```yaml
# Allow reading pods but nothing else in the 'app' namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: app
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: app
subjects:
- kind: ServiceAccount
  name: monitoring-sa
  namespace: app
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### Audit RBAC

```bash
# See what a service account can do
kubectl auth can-i list pods --as=system:serviceaccount:app:monitoring-sa

# Find overprivileged service accounts
kubectl get clusterrolebindings -o json | jq '.items[] | select(.roleRef.name=="cluster-admin")'
```

---

## NetworkPolicy: Restricting Pod Communication

By default, all pods can communicate with all other pods. NetworkPolicies restrict this.

```yaml
# Deny all ingress to pods in the 'backend' namespace
# except from pods with label 'role: frontend'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-only
  namespace: backend
spec:
  podSelector: {}    # applies to all pods in namespace
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
```

NetworkPolicies require a CNI plugin that supports them (Calico, Cilium, Weave). Minikube uses kindnet which does not enforce them by default - use Calico for real enforcement.

---

## Pod Security: securityContext

Set security constraints on individual containers:

```yaml
spec:
  securityContext:
    runAsNonRoot: true           # fail if container wants to run as root
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: app
    image: my-app:1.0
    securityContext:
      allowPrivilegeEscalation: false   # cannot gain more privileges
      readOnlyRootFilesystem: true      # cannot write to /
      capabilities:
        drop:
        - ALL                           # drop all Linux capabilities
        add:
        - NET_BIND_SERVICE              # only add what you need
```

### Why Each Setting Matters

| Setting | What it prevents |
|---------|-----------------|
| `runAsNonRoot` | Container running as UID 0 (root), which maps to host root |
| `readOnlyRootFilesystem` | Attacker writing malware or modifying the container |
| `allowPrivilegeEscalation: false` | SUID binaries escalating to root |
| `capabilities: drop: ALL` | Container using dangerous kernel capabilities |

---

## Secrets: Don't Put Them in Environment Variables

Kubernetes Secrets are base64-encoded (not encrypted) by default. Anyone with read access to the namespace can decode them.

Better approaches:
1. Enable **Secrets encryption at rest** in the API server
2. Use **Vault** or **AWS Secrets Manager** with a CSI driver
3. Use **External Secrets Operator** to sync from a secrets manager

```bash
# See all secrets in a namespace (if you have access - attackers will try this)
kubectl get secrets -n production
kubectl get secret db-credentials -o jsonpath='{.data.password}' | base64 -d
```

---

## Auditing with kube-bench

kube-bench checks your cluster against the CIS Kubernetes Benchmark:

```bash
# Run against minikube
docker run --rm --pid=host \
  -v /etc:/etc:ro \
  -v /var:/var:ro \
  -v /proc:/proc:ro \
  --net=host \
  aquasec/kube-bench \
  --benchmark cis-1.8
```

Each finding is PASS, FAIL, or WARN with a remediation step.

---

## Image Scanning with Trivy

```bash
# Scan an image
trivy image nginx:latest

# Only show CRITICAL and HIGH
trivy image --severity CRITICAL,HIGH nginx:latest

# Scan a running Kubernetes cluster
trivy k8s --report summary cluster
```

---

## Common Attack Techniques to Know

| Technique | How it works | Mitigation |
|-----------|-------------|------------|
| Exposed API server | kubectl accessible without auth | Require authentication, use kubeconfig |
| Privileged container | Container has host-level access | `privileged: false` in securityContext |
| hostPath mount | Container mounts host filesystem | Restrict with PodSecurity admission |
| Service account token abuse | Default token has too much access | Use minimal RBAC, disable auto-mount |
| Writable node filesystem | Container writes to node via hostPath | Deny hostPath mounts |
| Lateral movement via DNS | Pods resolve each other by name | Use NetworkPolicy to restrict |

---

## Key Things to Remember

- Default Kubernetes is not secure - it requires deliberate hardening
- RBAC least privilege: only grant the verbs a service account actually needs
- NetworkPolicy default-deny is the safest posture
- Never run containers as root - always set `runAsNonRoot: true`
- Scan images before deploying them - not after

---

## Activity: Harden a Kubernetes Cluster

1. Run kube-bench against your minikube cluster: docker run --rm --pid=host -v /etc:/etc:ro -v /var:/var:ro aquasec/kube-bench -- review the FAIL items
1. Create a ServiceAccount with least-privilege RBAC: write a Role that only allows get/list on Pods, bind it to a ServiceAccount
1. Apply a NetworkPolicy that blocks all ingress to a pod except from pods with a specific label
1. Run a pod with a read-only root filesystem: add securityContext.readOnlyRootFilesystem: true to a pod spec
1. Use Trivy to scan a running pod's image: trivy image nginx:latest -- note the CVEs
1. Read the Kubernetes attack matrix from Microsoft (search 'Microsoft Kubernetes attack matrix') -- identify 3 techniques you have now mitigated
1. You know you are done when you have applied RBAC, a NetworkPolicy, and a securityContext to a running pod

---

## Check-in Questions

Write your answers down before your next session.

1. What is RBAC in Kubernetes and what problem does it solve?
2. Why is running a container as root inside a pod a security risk?
3. What does a NetworkPolicy do and what does it not do?
4. What is a kubeconfig file and why would leaking one be catastrophic?
