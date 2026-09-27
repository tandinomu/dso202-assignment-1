# DSO202 Assignment 1: Three-Tier App on Kubernetes

## Introduction

This repository contains Kubernetes manifests for deploying a three-tier Task Tracker application on a local `kind` cluster. The application includes a frontend, backend, and database. The images were provided, so no application code was written.

The main focus was on configuring, connecting, securing, and managing the three tiers using Kubernetes.

## Task 1: Architecture Note

### Control Plane

The API server receives commands from `kubectl`. etcd stores the cluster state, the scheduler assigns pods to nodes, and the controller manager ensures that the required number of replicas keeps running. If a pod is deleted, the controller helps recreate it.

### Node

The kubelet manages the containers on each node. containerd runs the provided application images, while kube-proxy handles network routing for Services.

### Objects Used for Each Tier

**Database:** A Deployment, PersistentVolumeClaim (PVC), and headless Service (`clusterIP: None`) were used. The PVC keeps data separate from the pod lifecycle. The headless Service provides internal network identity for the database.

**Backend:** A Deployment and ClusterIP Service were used. The backend is accessible only within the cluster and is not exposed externally.

**Frontend:** A Deployment and NodePort Service were used. This allows the frontend to be accessed through a browser.

## Task 2: Secrets

The `configmap.yaml` file stores non-sensitive configuration values, while `secret.yaml` stores database credentials.

The credentials include `DB_USER`, `DB_PASSWORD`, `POSTGRES_USER`, and `POSTGRES_PASSWORD`.

The backend uses the `DB_*` variables, while the PostgreSQL image uses the `POSTGRES_*` variables. Both are configured with matching values.

## Task 6: Quota Justification

The following ResourceQuota and LimitRange were configured.

```text
ResourceQuota: pods=4, requests.cpu=1, requests.memory=1Gi,
               limits.cpu=2, limits.memory=2Gi

LimitRange (per container): default request 100m CPU/128Mi memory,
                            default limit   500m CPU/256Mi memory
```

The application normally runs three pods, one for each tier. The quota allows a fourth pod so that a rolling restart is less likely to be blocked by the pod limit.

The CPU and memory requests and limits provide resources for each tier while keeping the total usage within the namespace quota.

The configuration was tested by restarting all three Deployments. Quota usage increased from `0` to `300m` CPU and `384Mi` memory, matching the default requests for three containers.

![Quota and LimitRange](evidence/quota-and-limitrange.png)

## Task 7: Evidence

### a. Full CRUD Cycle

The complete CRUD cycle was tested using curl through a port-forwarded backend.

```bash
kubectl port-forward svc/backend-svc 8080:8080
```

**Create**

![Create](evidence/crud-01-create.png)

**List**

The list operation confirmed that the task was saved.

![List](evidence/crud-02-list.png)

**Update**

![Update](evidence/crud-03-update.png)

**Delete**

The task was deleted and its removal was confirmed.

![Delete](evidence/crud-04-delete.png)

### Testing Through the Frontend

For the UI demonstration, `BACKEND_URL` was temporarily set to `http://localhost:8080` using the backend port forward.

This was necessary because the browser cannot resolve the cluster-internal address `backend-svc`. The original configuration was restored after testing.

**Create**

![UI create](evidence/ui-01-create.png)

**Update**

![UI update](evidence/ui-02-update.png)

**Delete**

![UI delete](evidence/ui-03-delete.png)

### b. Service DNS Resolution

The backend Service was tested from inside the frontend pod using `kubectl exec`.

```text
http://backend-svc:8080/api/status
```

The request returned a healthy response, confirming that Kubernetes DNS could resolve the backend Service by name.

![DNS resolution](evidence/dns-resolution.png)

### c. Self-Healing and Data Persistence

The backend pod was manually deleted to test self-healing. The ReplicaSet automatically created a replacement pod.

A task created before deleting the pod was retrieved successfully after the replacement pod started.

**Task before deletion**

![Before deletion](evidence/selfheal-01-before-deletion.png)

**Pod automatically recreated**

![Pod recreated](evidence/selfheal-02-pod-recreated.png)

**Task retrieved after recreation**

![Data persisted](evidence/selfheal-03-data-persisted.png)

### d. Declarative and Imperative Comparison

A demo ConfigMap was created using both declarative and imperative methods.

**Declarative method**

```bash
kubectl apply -f demo-configmap.yaml
```

![Declarative](evidence/declarative.png)

**Imperative method**

```bash
kubectl create configmap ... --from-literal=...
```

![Imperative](evidence/imperative.png)

**Comparison**

| Feature | Declarative | Imperative |
|---|---|---|
| How it works | Desired state is written in a YAML file and applied | Resource is created directly using a command |
| Repeatability | Reapplying the same configuration is safe | Repeating the create command may fail if the resource already exists |
| Version control | Files can be stored and reviewed in Git | No configuration file is created automatically |
| Speed | Requires preparing a file and applying it | Faster for quick tasks |
| Best use | Reusable resources and application deployments | Quick testing and temporary resources |

Declarative management was used for the main application resources because the YAML files can be reused, reviewed, and stored in version control. Imperative commands are useful for quick tasks and debugging.

## Problems Faced

- The original three-node `kind` cluster was lost after a restart. A new cluster was created, and port forwarding was used when needed.
- The database image took a long time to download. The `kind load docker-image` command was used to load it directly into the cluster.
- The backend and frontend images did not include curl. The JavaScript `fetch()` function and `wget` were used to test connectivity instead.
- The browser could not access the internal `backend-svc` address. Port forwarding was used to test the frontend.
- Some port-forwarding commands failed because older processes were still running. These processes were identified using `lsof` and stopped using `kill`.

## Conclusion

All three tiers were successfully deployed and connected using Kubernetes Services, ConfigMaps, and Secrets. Persistent storage, self-healing, resource limits, CRUD operations, and internal DNS resolution were tested.

The practical demonstrated how Kubernetes components work together to deploy, connect, and manage a multi-tier application.

## Deploy

Run the following commands in order to deploy the application.

```bash
kubectl apply -f namespace.yaml
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
kubectl apply -f quota.yaml
kubectl apply -f database/
kubectl apply -f backend/
kubectl apply -f frontend/
```