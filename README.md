# DSO202 — Assignment 1: Three-Tier App on Kubernetes

Deploys a pre-built Task Tracker (frontend, backend, database) into the
`dso202-assignment-01` namespace on a local kind cluster. All manifests
are in this repo, split by tier.

## Task 1 - Architecture Note

**Control plane:** API server receives `kubectl` commands, etcd stores
the state, the scheduler places each pod on a node, and the controller
manager keeps each Deployment's replicas running (this is what recreates
a pod if it's deleted).

**Node:** kubelet starts/stops the containers, containerd runs the
tutor-provided images, kube-proxy routes traffic for each Service.

**Objects per tier:**
- **Database** - Deployment + PVC + headless Service (`clusterIP: None`).
  PVC keeps data independent of the pod's lifecycle; headless because
  there's one instance and it must stay internal-only.
- **Backend** - Deployment + ClusterIP Service. Internal-only, never
  exposed outside the cluster.
- **Frontend** - Deployment + NodePort Service. The one tier that needs
  to be reachable from a browser.

## Task 2 - Secrets

`configmap.yaml` holds non-sensitive values. `secret.yaml` holds
credentials (`DB_USER`, `DB_PASSWORD`, `POSTGRES_USER`,
`POSTGRES_PASSWORD`). The backend uses `DB_*` names, the official
Postgres image uses `POSTGRES_*` names — both are set to matching values.

**Note:** Kubernetes Secrets are only base64-encoded, not encrypted at
rest by default. Anyone with API access can decode them
(`echo <value> | base64 -d`), so this is not real secret protection.

## Task 6 - Quota Justification

```
ResourceQuota: pods=4, requests.cpu=1, requests.memory=1Gi,
               limits.cpu=2, limits.memory=2Gi
LimitRange (per container): default request 100m CPU/128Mi memory,
                             default limit   500m CPU/256Mi memory
```

3 tiers × 1 replica = 3 pods normally; the quota allows a 4th so a
rolling restart isn't blocked. Requests/limits are sized at roughly 3×
the LimitRange default, giving each tier a guaranteed baseline plus
headroom to burst without starving the others. Verified by restarting
all three Deployments and confirming quota usage rose from `0` to
`300m` CPU / `384Mi` memory (3 × the default request).

![Quota and LimitRange](evidence/task6-quota-limitrange.png)

## Task 7 - Evidence

### a. Full CRUD Cycle
Performed via `curl` through a port-forwarded backend
(`kubectl port-forward svc/backend-svc 8080:8080`).

**Create:**
![Create](evidence/task7a-01-create.png)

**List (confirming it was persisted):**
![List](evidence/task7a-02-list.png)

**Update:**
![Update](evidence/task7a-03-update.png)

**Delete (and confirmation it's gone):**
![Delete](evidence/task7a-04-delete.png)

### b. Service DNS Resolution
From inside the frontend pod (`kubectl exec`), curled the backend
Service by name (`http://backend-svc:8080/api/status`) and got a
healthy response — confirms cluster DNS resolves Services correctly.

![DNS resolution](evidence/task7b-dns-resolution.png)

### c. Self-Healing & Data Persistence
The backend pod was deleted manually; the ReplicaSet recreated it
automatically; a task created beforehand was still retrievable
afterward.

**Task exists before deletion:**
![Before deletion](evidence/task7c-01-before-deletion.png)

**Pod deleted and automatically recreated by the ReplicaSet:**
![Pod recreated](evidence/task7c-02-pod-recreated.png)

**Same task still retrievable after the new pod came up:**
![Data persisted](evidence/task7c-03-data-persisted.png)

### d. Declarative vs. Imperative Comparison
A demo ConfigMap was created both ways.

**Declarative** (`kubectl apply -f demo-configmap.yaml`):
![Declarative](evidence/task7d-01-declarative.png)

**Imperative** (`kubectl create configmap ... --from-literal=...`):
![Imperative](evidence/task7d-02-imperative.png)

**Comparison:**

| | Declarative (`kubectl apply -f`) | Imperative (`kubectl create ...`) |
|---|---|---|
| **How it's done** | Desired state written to a YAML file, then applied | Object created directly via a single command |
| **Repeatability** | Idempotent — reapplying the same file is always safe | Not idempotent — running it again fails or errors if the object exists |
| **Version control** | File can be committed, reviewed, and diffed in Git | No file produced — nothing to commit or review |
| **Speed** | Slightly slower (write file, then apply) | Faster for quick, one-off objects |
| **Best for** | Real, reusable resources (used for everything else in this assignment) | Throwaway objects or quick debugging |

## Deploy

```bash
kubectl apply -f namespace.yaml
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
kubectl apply -f quota.yaml
kubectl apply -f database/
kubectl apply -f backend/
kubectl apply -f frontend/
```