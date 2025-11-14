# Docker Containers — Deep Dive

## Agenda

- What exactly is a container — a deep dive

---

## 1. Why Containers Are Ephemeral

A container is simply:

- **A Linux process** + **a temporary root filesystem**.

The filesystem layers (image layers + writable layer) exist **only while the container runs**.

**Key point:**  
If the main container process exits → container stops → writable layer disappears → the container is gone.

This makes containers **stateless and ephemeral by design**.

---

## 2. Container = Just a Process on the Host

When you run:

```bash
docker run nginx
```

Docker does **not** start a virtual machine. It launches a **Linux process on the host**, but inside **kernel namespaces**.

**Diagram:**

```
Host OS
│
├── systemd (PID 1)
├── sshd
├── containerd
│   └── containerd-shim
│       └── nginx   <-- THIS IS THE CONTAINER!
```

The "container" is simply the **nginx process** running under isolation.

---

## 3. Why Container Isolation = Pseudo-Isolation

- **Namespaces** hide parts of the world.
- **Cgroups** limit resource usage.

**But:**

- All containers share the **same host kernel**.
- All container processes appear in the host's `ps -ef`.
- A root user inside the container is **not** real root (User Namespace).
- A process inside a container can't break out normally, but under kernel vulnerabilities it can escape — because it's not a VM.

**Thus:**

- **Containers ≠ Virtual Machines**
- **Containers = Regular processes with illusions**

That's why we call it **process-level isolation**, not OS-level isolation.

---

## 4. Understanding containerd, shim, runc

Modern container architecture:

```
docker --> containerd --> shim --> runc --> your process
```

| Component | Role |
|-----------|------|
| **containerd** | Long-running daemon. Manages containers. Handles images, snapshots, networking. |
| **containerd-shim** | One shim per running container. Acts as parent for container process. Allows containerd to restart without killing containers. |
| **runc** | The actual runtime. Uses Linux kernel syscalls (`clone`, `unshare`, `pivot_root`). Sets up namespaces & cgroups. Starts the main process inside the new isolated environment. |

**Diagram:**

```
+--------------------------+
| docker client | Docker Engine (API) |
+-----------+--------------+
            |
            v
+--------------------------+
| containerd               |
+-----------+--------------+
            |
            (one per container)
+-----------------------+
| containerd-shim       |
+-----------+-----------+
            |
            v
+-----------+
| runc      |
+-----+-----+
      |
      v
+---------------------+
| Your container app  |
| (e.g., nginx)       |
+---------------------+
```

---

## 5. Process Tree Explains Container Behavior

- A container is started by **containerd-shim**.
- Shim launches **runc**.
- **runc** sets up namespaces.
- **runc** starts the container's main process (e.g., `/usr/sbin/nginx`).

**Process tree example:**

```bash
$ pstree -pl
containerd(233)
 └── containerd-shim(9421)
     └── nginx(9422)
         ├── nginx worker(9423)
         └── nginx worker(9424)
```

**Important:**  
The main process inside the container becomes **PID 1** in the container's PID namespace. But on the host, it is just another process (e.g., 9422).

---

## 6. Why Killing the Main Process Stops the Container

Because: **The container lifecycle = lifecycle of its PID 1**.

If PID 1 exits in the container namespace:

1. containerd-shim detects process exit
2. Shim notifies containerd
3. containerd marks container as stopped
4. Writable layer is discarded
5. Networking is removed
6. Cgroups cleaned
7. Container disappears

**Example:**

```bash
docker kill <container>
```

This sends SIGKILL to PID 1. Container stops instantly.

---

## 7. Why Containers Are Ephemeral (with diagrams)

Each container has:

- A **read-only** image
- A **temporary writable** layer

**Diagram:**

```
Image Layers (read-only)
  layer 1
  layer 2
  layer 3
-------------------------
Container Writable Layer (deleted on stop)
  /var/log/app.log
  /tmp/files
-------------------------
Running Process
```

When container stops → writable layer is **deleted** → all state **lost**.

**Thus:** Container data disappears unless stored in:

- **Volumes**
- **Bind mounts**
- **External storage** (DB, S3, etc.)

---

## 8. Why Containers Appear Like Isolated Machines

**Namespaces** create illusions:

| Namespace | Effect |
|-----------|--------|
| **PID** | Container process sees itself as PID 1. |
| **Network** | Container sees its own eth0, IP address, routing table. |
| **Mount** | Container sees its own root filesystem. |
| **UTS** | Container sees its own hostname. |
| **IPC** | Own shared memory. |
| **User** | Root inside container ≠ root on host. |

---

## 9. Why Containers Start So Fast

- No kernel boot
- No virtual hardware
- No BIOS
- No init system (unless you add one)

Containers start a **single process**:  
`runc` → `clone` syscall → process runs.

- **Container start time:** ~50ms  
- **VMs:** tens of seconds

---

## 10. Why Containers Are Light

Containers share:

- The host kernel
- The host OS
- Libraries (optional)
- CPU scheduler
- Memory management

Only the **filesystem** + **process isolation** is unique.

---

## 11. Why Containers Fail If the Main Process Exits

A container = **one main process**.

Examples:

- `nginx` dies → container dies
- `python app.py` exits → container exits
- `sleep 5` completes → container exits immediately

This confuses beginners. Containers often need:

- **Process managers** (e.g., supervisord), or
- **Multi-process apps** designed properly

---

## 12. Summary in 10 Bullet Points

1. A container is just a **process + isolation**.
2. It runs on the **host OS**, not in a VM.
3. Isolation is via **namespaces**.
4. Resource limits via **cgroups**.
5. Process started by **runc**, managed by **containerd-shim**.
6. The container's root process is **PID 1** in its namespace.
7. If **PID 1 dies** → container stops.
8. **Writable layer** is temporary → ephemeral.
9. Isolation is not strong like a VM → **"pseudo-isolation"**.
10. All container processes visible in host `ps -ef`.

---

# Containers Explained — From Zero to Deep Internals

## 1. What EXACTLY Is a Container?

A container is **not:**

- a virtual machine  
- a mini-OS  
- something magical  

A container **is:**

- **A normal Linux process** with special isolation applied using kernel features.

**Why does it feel like a separate computer?**

Because it has its own:

- User space  
- IPC (Inter-Process Communication) space  
- Filesystem  
- Network interface  
- Hostname  
- Process tree  
- Resource limits  

But none of this is done by creating a new OS. All of it is done using:

- **Linux Namespaces**
- **Linux Cgroups**
- **Chroot / Filesystem overlays**
- **Capabilities**
- **Seccomp** (optional)

---

## 2. Why a Container Is Just a Process

**Normally**, when you run a process:

```bash
$ python app.py
```

the process shares: host network (eth0), host filesystem, host PID space, host users, host resources.

**In a container:**

```bash
docker run python:3.11
```

Docker creates a new process (e.g. via `containerd-shim-runc-v2`) and runs `python` inside it — but it's still a **Linux process on the host**.

Before launching it, Docker enables isolation:

- PID namespace  
- NET namespace  
- UTS namespace  
- MOUNT namespace  
- USER namespace  
- IPC namespace  

Because of these namespaces, the process sees a **fake world**.

---

## 3. The Magic: Linux Namespaces

Think of namespaces like **AR (Augmented Reality) goggles** — each namespace puts a filter on what the process can see.

### 3.1 PID Namespace (Process ID Isolation)

**Without namespace — host processes:**  
1 systemd, 22 sshd, 300 nginx, 4212 python  

**Inside a container:**

```
/ # ps
PID  Command
  1  python
  7  bash
 12  app
```

The container sees its own process as **PID 1**. That makes it feel like its own OS.

**Diagram:**

```
+------------------ Host PID namespace -------------------+
| 1 systemd  22 sshd  300 nginx  4212 python(container)   |
+----------------------------------------------------------+
            |
            | <--- PID namespace isolation
            v
+--------------- Container PID namespace ---------------+
| PID 1 = python   PID 7 = bash   PID 12 = app          |
+-------------------------------------------------------+
```

### 3.2 Network Namespace

Each container gets its own:

- Network stack  
- IP address  
- Routing table  
- Firewall rules  

The host only sees a **veth pair**.

**Host:** `veth0 <======> docker0 bridge`  

**Inside container:** `eth0 -> 172.17.0.2`  

That's why containers have their own IPs.

### 3.3 Mount Namespace (Filesystem Isolation)

Each container sees its **own root filesystem** (`/`). Under the hood Docker uses:

- OverlayFS layers  
- Chroot jail  
- Bind mounts for volumes  

**Host filesystem:** `/` → bin, usr, home  
**Container filesystem:** `/` (overlay) → bin, usr, app (your files)  

Different root = feels like a different machine.

### 3.4 UTS Namespace (Hostname Isolation)

Container can have its own **hostname** and **domain name**.  
e.g. inside container: `hostname = web-app`; on host: `hostname = ip-10-0-0-12`.

### 3.5 IPC Namespace (Shared Memory Isolation)

Containers get their own semaphores and shared memory segments.

### 3.6 User Namespace

Containers can run as **root inside** the container but be **mapped to a non-root UID** on the host.

---

## 4. Cgroups — Controlling Resources

- **Namespaces** = isolation  
- **Cgroups** = resource limits  

Cgroups limit: **CPU**, **Memory**, **Disk I/O**, **PIDs**.

Example: limit container to 256MB RAM and 0.5 CPU. If you run too many processes, cgroups prevent the container from using more than allowed.

**Diagram:**

```
+----------------- Host Resources -----------------+
| CPU: 8 cores   RAM: 32GB   I/O: 1GB/s           |
+--------------------------------------------------+
     |              |              |
     v              v              v
+----------+  +-----------+  +------------------+
| cgroup A |  | cgroup B  |  | cgroup C         |
| CPU 1    |  | CPU 0.5   |  | CPU 0.2          |
| RAM 2GB  |  | RAM 256MB |  | RAM 128MB        |
+----------+  +-----------+  +------------------+
```

---

## 5. How Does Docker Create a Container?

Step-by-step:

1. Pull an image  
2. Unpack filesystem layers (OverlayFS)  
3. Create cgroups for limits  
4. Create namespaces  
5. `chroot` into the new root filesystem  
6. Start the process (e.g., python)  

So `docker run nginx` is effectively:

- create-net-namespace  
- create-mount-namespace  
- create-pid-namespace  
- apply-cgroups  
- chroot to overlayfs  
- launch `/usr/sbin/nginx`  

---

## 6. Why Normal Programs Do NOT Feel Like Containers

Normal programs:

- Share host network  
- Share host filesystem  
- Share host hostname  
- Share host PID tree  
- Share all CPU/RAM  

Containers **hide** all of that.

---

## 7. VM vs Container (Deep Difference)

**VM architecture:**

```
Hardware
  │
  Hypervisor
  │
  Guest OS (Linux/Windows)
  │
  Your App
```

**Container architecture:**

```
Hardware
  │
  Linux Kernel
  │
  Your App (isolated using namespaces + cgroups)
```

Docker does **not** run a separate kernel — only **processes**.