# Class Revision Notes: AWS and Terraform Concepts

*Comprehensive revision of concepts from the live class — AWS usage and Terraform principles.*

---

## Key Concepts Covered

### 1. Subnets and Their Dependencies

**Subnets** — In AWS, a **subnet** is a range of IP addresses within your **VPC**. There are **public** and **private** subnets.

| Type | Description |
|------|--------------|
| **Private subnets** | Not directly reachable from the internet. They do **not** have a route to the **internet gateway**. |
| **Public subnets** | Connected to the **internet gateway** and can be accessed from the internet. |

**Implicit dependencies in Terraform**

- Terraform infers **resource dependencies** from references between resources (e.g. one resource uses another’s ID).
- By referencing a resource (e.g. an AWS subnet) in another resource block, Terraform determines the **order of creation** automatically.

---

### 2. VPC and Network Configuration

**Virtual Private Cloud (VPC)** — A **VPC** is a virtual network dedicated to your AWS account, logically **isolated** from other networks in AWS.

**CIDR blocks**

- **CIDR** (Classless Inter-Domain Routing) defines how IP addresses are grouped for networks.
- When you create a VPC, you set its **CIDR block** to define the **size and IP range** of the VPC.

**Routing tables**

- Define **how traffic is routed** inside the VPC.
- They direct traffic based on **destination IP** (e.g. to subnets, internet gateway, NAT gateway).

---

### 3. Terraform Basics

**State management**

- Terraform uses a **state file** to track the infrastructure it manages.
- The file **`terraform.tfstate`** stores the current state of your managed resources.
- For **multiple users** or **shared infrastructure**, use **remote state** to avoid conflicts:
  - **AWS S3** — store the state file.
  - **DynamoDB** — **state locking** so only one run applies at a time.
- Remote state keeps everyone on the same state and reduces discrepancies.

**Common Terraform file layout**

| File | Typical contents |
|------|--------------------|
| **main.tf** | Primary Terraform configuration; core resources and providers. |
| **network.tf** | Network resources: VPC, subnets, internet gateway, routing. |
| **compute.tf** | Compute resources: EC2 instances, security groups. |

---

### 4. Types of IPs

| Type | Description |
|------|--------------|
| **Private IP** | Assigned to resources inside a VPC. Reachable only **within that VPC**. |
| **Public IP** | Assigned to resources that need internet access. Reachable **from outside** AWS. |
| **Elastic IP** | A **persistent** public IP tied to your AWS account. Can be **attached/detached** from instances (e.g. EC2). |

---

### 5. Launching EC2 Instances

- You **cannot** create an EC2 instance without an **Amazon Machine Image (AMI)**.
- An **AMI** defines the **OS and initial software** for the instance.
- AWS provides AMIs for various operating systems; you choose the right AMI when defining the EC2 resource in Terraform (or in the console).

---

### 6. Variable Management in Terraform

Terraform supports **variables** to make configs reusable and clearer.

| Type | Purpose |
|------|---------|
| **Input variables** | Make configurations **customizable** (e.g. region, instance type, environment). |
| **Output variables** | **Expose** information about the infrastructure (e.g. instance ID, public IP, endpoint). |
| **Local variables** | **Limit scope** to a single configuration file; avoid repetition inside that file. |

---

## Summary

| Topic | Takeaway |
|-------|----------|
| **Subnets** | Public = route to internet gateway; private = no direct internet. Terraform infers creation order from references. |
| **VPC** | Isolated virtual network; CIDR defines size/range; routing tables direct traffic. |
| **Terraform state** | `terraform.tfstate` tracks resources; use S3 + DynamoDB for remote state and locking. |
| **File layout** | main.tf, network.tf, compute.tf — separate concerns for clarity. |
| **IPs** | Private (VPC-internal), public (internet), Elastic (persistent public). |
| **EC2** | AMI is required to launch an instance. |
| **Variables** | Input (customize), output (expose), local (scope to file). |

These concepts are fundamental for **structuring AWS resources with Terraform**, managing dependencies, handling IPs, and keeping infrastructure **organized and maintainable** in DevOps and cloud workflows.