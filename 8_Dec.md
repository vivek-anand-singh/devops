# Kubernetes Services — Detailed Tutorial

## Why Services Exist

Pods in Kubernetes are **ephemeral**:

- Get **new IPs** on restart
- Can be **recreated** by ReplicaSets
- Can **scale up/down** dynamically

You **cannot** reliably communicate with Pods directly by IP.  
**Services** provide a **stable virtual IP** (cluster-wide) and **load balancing** across Pod replicas.

---

## 1. What Is a Kubernetes Service?

A **Service** is an abstraction that provides:

- A **stable virtual IP** (ClusterIP)
- A **stable DNS name**
- **Load-balancing** across backend Pods
- **Traffic routing** through kube-proxy (iptables/ipvs)
- Access from **inside or outside** the cluster

---

## 2. How a Service Works Internally

1. You create a Service with a **pod selector**:  
   `selector: app: myapp`
2. **kube-proxy** creates load-balancing rules using:
   - **iptables** (older, common)
   - **IPVS** (newer, faster)
3. Service gets a **stable ClusterIP**.
4. Any Pod contacting that ClusterIP goes through **kube-proxy**, which **forwards traffic** to matching Pods.

---

## 3. Types of Kubernetes Services

There are **3** main service types:

| Service Type   | Purpose                               | Accessible From        |
|----------------|----------------------------------------|------------------------|
| **ClusterIP** (default) | Internal communication                 | Inside the cluster     |
| **NodePort**   | Expose service on each node's IP       | External (nodeIP:nodePort) |
| **LoadBalancer** | Expose via cloud LB (AWS ELB/ALB/NLB, GCP, Azure) | Internet or VPC    |

---

## ClusterIP Service (Default)

Most common type — used for **internal** service-to-service communication.

**Use cases:**

- Microservices calling each other
- Internal databases
- Backends not exposed to the internet
- Frontend → backend, backend → database

### ClusterIP YAML Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - port: 80        # Service port
      targetPort: 8080 # Pod containerPort
```

**Breakdown:**

- **port:** Port exposed by the Service
- **targetPort:** Pod's container port
- **ClusterIP:** Assigned automatically

**How to access (inside cluster):**

```bash
curl http://backend-svc
```

Or with full DNS:

```bash
curl http://backend-svc.default.svc.cluster.local
```

---

## NodePort Service

Exposes a Service on **every node's IP** at a **static port** (NodePort).

- **NodePort range:** 30000–32767

**Traffic flow:**

```
External Client → NodeIP:NodePort → Service → Pods
```

**Use cases:**

- Non-cloud environments (bare-metal)
- Local clusters (kubeadm, kind, minikube)
- Debugging external access

### NodePort YAML Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-nodeport
spec:
  type: NodePort
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 32080   # Optional; can be auto-assigned
```

**Access externally:**

```
http://<NodeIP>:32080
```

You can use **any worker node IP** — Kubernetes forwards to the right Pods.

**NodePort limitations:**

- Exposes service on **all nodes** → not ideal for security
- Fixed high port range
- No smart routing (no CDN, health checks)
- Not production-grade for public internet

---

## LoadBalancer Service (Cloud Only)

Used to expose a Service **publicly** using a **cloud load balancer**.

**Supported in:** AWS (ELB, NLB, ALB), GCP, Azure, DigitalOcean

**Traffic flow:**

```
Internet → Cloud Load Balancer → NodePort → Service → Pods
```

Internally, LoadBalancer **still uses NodePort**.

### LoadBalancer YAML Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-lb
spec:
  type: LoadBalancer
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
```

**After creation:**

```bash
kubectl get svc backend-lb
```

You will see **EXTERNAL-IP:** public IP assigned by the cloud provider.

**Access:** `http://<EXTERNAL-IP>`

---

## Additional Concepts

### Selectorless Services (Headless Services)

Set:

```yaml
clusterIP: None
```

**Used for:**

- StatefulSets
- Databases
- DNS-based discovery

### Multi-Port Services

A single Service can expose **multiple ports**:

```yaml
ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: https
    port: 443
    targetPort: 8443
```

### Service DNS Resolution

Every Service gets DNS:

```
<servicename>.<namespace>.svc.cluster.local
```

**Example:** `redis.master.svc.cluster.local`

### Service Without Selector

**Used when:**

- Manually managing Endpoints
- **ExternalName** services

**Example (ExternalName):**

```yaml
kind: Service
spec:
  type: ExternalName
  externalName: database.company.com
```

---

## Which Service Type to Use?

| Need                         | Service Type   |
|-----------------------------|----------------|
| Internal-only microservices | **ClusterIP**  |
| Expose to local network without cloud LB | **NodePort**  |
| Production external access | **LoadBalancer** |
| StatefulSets / DBs          | **Headless** (clusterIP: None) |
| Access external DB          | **ExternalName** |