# Kubernetes Pod Notes

## 1. Quick Summary (One-Liner)

A **Pod** is the **smallest deployable unit** in Kubernetes: one or more **co-scheduled containers** that share **network namespace**, **IPC**, and **volumes**. Creation flows from the user/client (kubectl) to the API server (may be mutated/validated), is persisted to **etcd**, **scheduled** by the scheduler (assigns a node), and then the **kubelet** on the chosen node observes the Pod object and drives the **container runtime** to create containers.

---

## 2. Control Plane & Data Plane — High-Level Roles (Short)

| Component | Role |
|-----------|------|
| **API Server** (kube-apiserver) | Single entry point for all REST operations; validates, authenticates, authorizes, runs admission controllers, stores objects in etcd, serves watch streams. |
| **etcd** | Consistent key-value store; persistent source of truth for cluster state (e.g. `/registry/...` keys). |
| **Controller Manager** (kube-controller-manager) | Runs controllers: ReplicationController/ReplicaSet, Deployment, Job, Node controller, Endpoint controller, ServiceAccount controller, etc. |
| **Scheduler** (kube-scheduler) | Assigns unscheduled Pods to Nodes (by writing `Pod.spec.nodeName` or creating a Binding). |
| **Cloud Controller Manager** | Cloud-provider specific controllers (routes, load balancers, node lifecycle in cloud). |
| **kubelet** (on each node) | Node agent: watches API server for Pod objects assigned to its node, starts/stops containers via CRI, reports status. |
| **Container Runtime** (CRI-O/containerd/dockerd) + OCI runtime (runc/crun) | Actually pulls images and creates containers. |
| **kube-proxy** | Programs networking (iptables/ipvs) for Services on each node. |
| **CNI plugins** | Set up Pod network interfaces and IPs. |
| **CSI drivers** | Attach/mount volumes on nodes when Pods request persistent volumes. |

---

## 3. Pod Object Basics (What Is Stored in etcd)

- A **Pod** is a group of **one or more containers**.
- All containers in the Pod are **co-located** and **co-scheduled**.
- A Pod object is a Kubernetes API resource (`api/v1`), typically stored under a path like:
  - `/registry/pods/<namespace>/<podName>`

**Key important fields:**

- `metadata.uid`, `metadata.resourceVersion`, `metadata.creationTimestamp`
- **spec** — containers, initContainers, volumes, nodeName (initially empty when unscheduled), tolerations, affinity, etc.
- **status** — phase, conditions, containerStatuses (updated by kubelet)

`resourceVersion` is used for **optimistic concurrency** and **watch** semantics.

---

## 4. Step-by-Step Creation Flow (Detailed Sequence)

### Sequence (Short)

1. `kubectl apply -f pod.yaml` → kubectl calls API server (REST).
2. API server authenticates/authorizes request.
3. Admission controllers run (mutating → validating). Mutations may change spec (e.g. inject sidecar).
4. API server persists Pod object to etcd (write).
5. API server returns created Pod object to client.
6. kube-scheduler sees unscheduled Pod (via watch or list).
7. Scheduler filters & scores nodes; chooses node → writes binding (sets `spec.nodeName` or creates a Binding).
8. API server persists that update to etcd. A watch event is emitted.
9. Kubelet on the chosen node sees Pod assigned to it (via watch/list) and enqueues it to the pod sync loop.
10. Kubelet creates the Pod: calls CRI to create sandbox (pause/infra container), then pulls images and creates containers.
11. Kubelet reports PodStatus back to API server (status subresource patch/write). etcd updated.
12. kube-proxy / CNI / CSI finalize network and volume attachments as needed; Services/endpoints updated; Pod becomes Ready when readyProbe passes.
13. Controllers (ReplicaSet/Deployment) continue to observe and reconcile desired state.

---

### Expanded Details

#### Step 1 — Client Submits Pod to API Server

```bash
kubectl apply -f mypod.yaml
# or
kubectl create -f mypod.yaml
```

kubectl performs an HTTP REST call to `POST /api/v1/namespaces/<ns>/pods` (or PATCH for apply). The request includes the Pod manifest in JSON/YAML.

#### Step 2 — API Server: authN / authZ

- **Authentication:** API server checks client identity (client cert / OIDC token / service account token).
- **Authorization:** RBAC or ABAC decides whether the caller can create the Pod.
- If auth fails, it returns **401/403** and the flow stops.

#### Step 3 — Admission Controllers (mutating → validating)

- **Mutating** Admission Webhooks & built-in mutating controllers run **first**. They can:
  - Inject defaults.
  - Inject sidecars (e.g. Istio, Linkerd) via MutatingAdmissionWebhook.
  - Add imagePullSecrets, default resource limits, security policies, etc.
- **Validating** Admission Webhooks & built-ins run **after** mutations and can **reject**.
- Common built-ins: NamespaceLifecycle, LimitRanger, MutatingAdmissionWebhook, ValidatingAdmissionWebhook, ResourceQuota, PodSecurity (or PodSecurityPolicies in older clusters).

**Important:** The object that gets persisted may differ from the user's original YAML because of mutations (e.g. a sidecar container added).

#### Step 4 — API Server Writes Pod to etcd

- The API server marshals the final Pod object into JSON and writes it to etcd under `/registry/pods/<namespace>/<name>`.
- etcd assigns a new modRevision. The Pod gets a `metadata.resourceVersion` that reflects storage version.
- The etcd write is **strongly consistent (linearizable)** — once the write returns success, other readers that request a current read will see it.
- Note: API server also keeps an in-memory cache and responds to other components (controllers/scheduler) via watches.

#### Step 5 — API Server Responds to the Client

kubectl receives success and prints summary. The Pod is **unscheduled** if `spec.nodeName` is not set.

#### Step 6 — Scheduler Notices Unscheduled Pod

- The scheduler watches the API for Pods where `spec.nodeName` is empty and `status.phase` is not failed/succeeded.
- **Two phases:**
  1. **Filtering (predicates):** Remove nodes that cannot run the pod (resource requests, taints & tolerations, nodeSelectors, affinity, node disk/capacity, port conflicts, volume constraints, topology).
  2. **Scoring:** Rate the remaining nodes (binpacking, spread, custom scheduler extenders, plugin scores).
- Scheduler may also consider **Pod priority and preemption**: if no node fits, it may preempt lower-priority pods to free resources.
- Scheduling decision results in a **Bind** operation.

#### Step 7 — Scheduler Writes the Binding / Sets spec.nodeName

Two ways (historically):

- **Create a Binding subresource:**  
  `POST /api/v1/namespaces/<ns>/pods/<pod>/binding` with  
  `{ "target": { "apiVersion": "v1", "kind": "Node", "name": "node-1" } }`  
  → API server sets `spec.nodeName` for the Pod.
- **Patch** the Pod object to set `spec.nodeName` (less common in modern schedulers).

This write goes through API server → etcd and is subject to admission. The write increments resourceVersion and emits a watch event for the Pod.

#### Step 8 — Watchers Receive the Event (Kubelet Sees Assignment)

- Kubelet(s) watch Pods (typically watch all Pods and filter by `spec.nodeName == myNode` locally).
- When kubelet sees a Pod assigned to its node, it **enqueues** it to the pod worker queue.
- There may be a slight delay between scheduler binding and kubelet seeing it due to watch propagation.

#### Step 9 — Kubelet's Pod Sync Loop (Create Pod)

Kubelet processes the Pod in its sync loop:

1. **Admission & validation at kubelet:** Checks Pod security (PSP/PSA), volumes, local node features. Uses CRI.
2. **Prepare volumes:** If volumes are required, kubelet coordinates with CSI/attach-detach controllers or mounts local volumes.
3. **Setup networking:** Calls CNI plugin to allocate Pod IP and set up veth pair / network namespace. CNI result (podIP) is stored in PodStatus.
4. **Create Pod sandbox:** Calls CRI `RuntimeService.RunPodSandbox` — creates the infra/pause container that holds the network namespace.
5. **Pull images:** Via CRI `ImageService.PullImage`.
6. **Create containers:** `RuntimeService.CreateContainer` for each container, then `StartContainer`.
7. **PostStart hooks / init containers:** Init containers run sequentially; postStart lifecycle hooks run inside the container runtime.
8. **Set status:** Kubelet writes PodStatus updates back to the API (status subresource). Status includes phase (Pending/Running/Failed), containerStatuses (image, state waiting/running/terminated), podIP.

Kubelet keeps working to ensure Pod remains desired (restarts failed containers per restartPolicy).

#### Step 10 — Readiness & Liveness Probes and Endpoints

- **Readiness probe:** When it passes, kubelet sets `containerStatuses[].ready = true`; Endpoints controller adds the Pod to Service endpoints. Until ready, the Pod doesn't receive Service traffic.
- **Liveness probe:** Failing liveness causes kubelet to restart the container per restart policy.
- These changes are reflected as Status updates in the Pod object in etcd.

#### Step 11 — Controllers & Services Notice Pod State

- ReplicaSet/Deployment observes Pod creation via informers and updates status.replicas. If more/fewer Pods than desired, the controller creates or deletes Pods.
- Service controllers add Pod IP to Endpoints/EndpointSlices as Pod becomes ready.

---

## 5. What Happens in etcd — Deeper Detail and Examples

etcd stores the **authoritative serialized object**. Typical key layout (simplified):

```
/registry/pods/<namespace>/<podName> -> Pod JSON (spec, status)
```

**Storage events:**

- **Create:** API server PUT creates key with createRevision & modRevision. Object contains spec and initial status (often empty except conditions).
- **Update (binding):** Scheduler writes nodeName → modRevision increments.
- **Status updates:** Kubelet patches the status subresource → status changes update resourceVersion again.

**Watches:** Components (scheduler, kubelet, controllers) maintain watches on etcd via API server; watch streams send events: ADDED, MODIFIED, DELETED with resourceVersion.

**Optimistic concurrency:** Writes include preconditions (e.g. resourceVersion match) so stale writes are rejected (**409**). Prevents races.

**Compaction & snapshot:** etcd compacts old revisions; regular backups/snapshots recommended. Leader election/leases (e.g. kube-controller-manager) use Lease objects in etcd.

**Example sequence:**

1. POST pod → etcd key `/registry/pods/ns1/pod1` created, resourceVersion 10.
2. Scheduler binds → update spec.nodeName → resourceVersion 12.
3. Kubelet status update → status.phase=Running, podIP=10.244.1.5 → resourceVersion 14.

Each update is an etcd write; watchers receive MODIFIED events.

---

## 6. Important Subresources & Concurrency Rules

| Concept | Rule |
|---------|------|
| **status subresource** | Kubelet updates status via `PATCH /status` so it doesn't overwrite spec (reduces conflicts). |
| **finalizers** | Used so controllers complete cleanup before deletion (e.g. volumes detach). |
| **resourceVersion** | Used for watches and optimistic concurrency. PUT with old resourceVersion → API server rejects (HTTP 409). |

---

## 7. Admission Webhooks Impact on Pod Creation

- **Mutating webhook** (e.g. sidecar injector) can mutate the object (add sidecar, mounts, env). The **mutated** object is what gets persisted. This can affect scheduling (resource requests) and kubelet behavior.
- **Validating webhook** can **reject** creation (e.g. disallow privileged containers). If validation fails, Pod creation is aborted — nothing written to etcd.
- Webhooks can run **synchronously** during creation (can delay API response until they return).

---

## 8. Networking / CNI / kube-proxy

- **CNI:** Kubelet calls CNI plugin to allocate an IP and configure networking in the Pod's netns (veth pair to node). CNI result provides **podIP**.
- **kube-proxy:** Creates Service routing rules in iptables/ipvs so Service IP → Pod IP translation works.
- **DNS:** kube-dns/CoreDNS watches Service & Endpoints and resolves service names to ClusterIP.

---

## 9. Storage / Volumes (CSI)

- If Pod requests a **PersistentVolume:** Controller may create volume via CSI controller (CreateVolume).
- **Attach/Detach** controllers ensure volume is attached to the node (for block volumes). Kubelet mounts the volume into the Pod filesystem via CSI **node** plugin (NodePublishVolume).
- Volumes can **delay** Pod starting until attach/mount completes.

---

## 10. Lifecycle Hooks, Init Containers, and Termination

- **Init containers** run **sequentially** before main containers; they must succeed, otherwise Pod stays in Init state.
- **PreStop** hooks run on graceful termination.
- **Graceful termination:** `kubectl delete pod` sets deletionTimestamp; kubelet runs termination logic, sends **SIGTERM**, waits for grace period, then **SIGKILL**.
- **finalizers** and **ownerReferences** ensure resources are cleaned up.

---

## 11. Pod States & Conditions (Where They Appear in etcd)

- **phase:** Pending, Running, Succeeded, Failed, Unknown.
- **conditions:** Ready, PodScheduled, Initialized, Unschedulable, etc. Set by controllers (scheduler sets PodScheduled, kubelet sets Ready and container statuses).
- **containerStatuses:** state.waiting / running / terminated, lastState, restartCount, image IDs.

---

## 12. Race Conditions & How Kubernetes Avoids Them

- **Optimistic concurrency** via resourceVersion.
- **Status subresource** to avoid spec/status clobbers.
- **Admission webhooks** can block conflicting changes early.
- **Leader election** among controllers via Lease objects in etcd.

---

## 13. Debugging — Commands, What to Look For, Where Things Commonly Fail

**Useful commands:**

```bash
kubectl get pods -A
kubectl describe pod <pod> -n <ns>   # events, scheduling decisions, reasons
kubectl get pod <p> -o yaml
kubectl get events -n <ns> --sort-by='.lastTimestamp'
kubectl logs <pod> -c <container>      # container logs
kubectl get endpoints <svc> -o yaml
kubectl get pods -o wide               # node assignment + IP
kubectl get nodes -o wide               # capacity
kubectl get replicaset,deploy -n <ns>

# Watch pending pods (scheduler view)
kubectl get pods --field-selector=status.phase=Pending -w
```

**Where things commonly fail:**

| Failure | What you'll see |
|---------|------------------|
| Auth/authZ | API returns 401/403; kubectl shows permission denied. |
| Admission webhook | kubectl shows webhook error; `describe pod` shows rejected reason. |
| Scheduling | Pod stays Pending; `describe pod`: "0/5 nodes available: ..." (Insufficient cpu/memory, nodeSelector, taints). |
| Image pull | containerStatuses[].state.waiting.reason = ImagePullBackOff — check image name, pullSecret, registry. |
| CNI failure | Pod stuck in ContainerCreating; events from CNI; check kubelet and CNI plugin logs. |
| Volume attach/mount | Pod stuck in ContainerCreating; attach/mount errors; check CSI controller and node plugin logs. |
| Kubelet issues | Node NotReady; `describe node` for conditions; kubelet logs. |
| CrashLoopBackOff | Check container logs and liveness probe. |

**Advanced debug:**

- Inspect etcd key (careful): `etcdctl get /registry/pods/<ns>/<pod> --write-out=...`
- Watch API events: `kubectl get events -w` or `kubectl get pod <pod> -w`

---

## 14. Example Trace — Realistic Short Trace (API Calls)

1. `kubectl apply -f pod.yaml` → `POST /api/v1/namespaces/default/pods` (body = Pod manifest).
2. API Server: authN → authZ → mutating webhook → validating webhook → write to etcd `/registry/pods/default/nginx-1`.
3. Response returned to kubectl (Pod created).
4. Scheduler: GET (or watch ADDED) unscheduled pods.
5. Scheduler decides node-05 → `POST /api/v1/namespaces/default/pods/nginx-1/binding` (target node-05).
6. API Server writes binding → etcd updated.
7. Kubelet on node-05 gets watch event MODIFIED → enqueues pod.
8. Kubelet: CNI allocate IP → CRI RunPodSandbox → CreateContainer (init + main) → StartContainer.
9. Kubelet patches status: `PATCH .../pods/nginx-1/status` with podIP and containerStatuses.
10. Pod transitions to Running and Ready after probe; Endpoints updated for Services.

---

## 15. Extras — Scheduler Plugins, Ephemeral Containers, Debug Hooks

- **Scheduler framework** supports plugins (permit, preFilter, filter, score, reserve, permit, preBind, bind, postBind). Custom schedulers or extenders can participate.
- **Ephemeral containers:** Can be added to an existing Pod for debugging (API subresource).
- **API aggregation:** Extension API servers can add new APIs; API server aggregates them.

---

## 16. Security & RBAC Notes

- API server enforces **RBAC** for who can create Pods.
- **Kubelet** enforces whether a pod can run privileged containers (PSA).
- **ServiceAccounts** issue tokens; controllers create Secrets (e.g. for image pull) mounted into Pods.

---

## 17. Best Practices (Pod Creation & Lifecycle)

- Use **Deployments/ReplicaSets/StatefulSets** to manage Pods (avoid raw Pods except for debugging).
- Define **resource requests and limits** for scheduling and to avoid contention.
- Use **readiness probes** so traffic isn't sent to not-yet-ready Pods.
- Use **liveness probes** to recover from unhealthy containers.
- Use **affinity/anti-affinity** and **taints/tolerations** for topology spread and isolation.
- Use **PodDisruptionBudget** to limit voluntary evictions.