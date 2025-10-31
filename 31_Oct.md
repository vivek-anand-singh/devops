# Engineering Master Notes: DevOps, AWS & Linux

## I. DevOps Introduction & Principles
* [cite_start]**Definition:** A set of practices and cultural philosophies that combine software development (Dev) and IT operations (Ops) to deliver services faster and more reliably[cite: 4, 134].
* [cite_start]**Core Goal:** To break silos between teams, automate repetitive tasks, and ensure continuous delivery[cite: 6, 7, 8].
* **The Lifecycle:** A continuous, iterative loop consisting of:
    * [cite_start]**Plan / Code / Build:** Requirement gathering and writing/compiling code[cite: 55, 136].
    * [cite_start]**Test / Release:** Automated QA and packaging code[cite: 55, 136].
    * [cite_start]**Deploy / Operate / Monitor:** Production releases and system health tracking[cite: 55, 136].



* [cite_start]**Key Principles:** Includes a culture of collaboration, Automation, CI/CD, and Infrastructure as Code (IaC)[cite: 60, 62, 64, 70, 137].

---

## II. AWS Fundamentals (Cloud Infrastructure)
* **Regions & AZs:**
    * **Region:** A geographically isolated area (e.g., us-east-1) containing multiple Availability Zones.
    * **Availability Zone (AZ):** Physically separate datacenters connected via low-latency links.
* **Networking (VPC):**
    * **VPC:** An isolated virtual network within a Region where you define IP ranges (CIDR) and subnets.
    * **Subnets:** Smaller network segments; "Public" subnets are internet-facing, while "Private" subnets require NAT Gateways.
* **EC2 Deployment Fields:**
    * **AMI:** The OS image (Ubuntu, Amazon Linux, etc.).
    * **Instance Type:** Determines CPU and RAM (e.g., t2.micro).
    * **Security Groups:** Virtual firewalls for inbound/outbound traffic.
    * **User Data:** Scripts that run at launch to configure the instance automatically.

---

## III. Linux Fundamentals
* **Architecture:** The **Kernel** manages hardware resources (CPU, Memory), while the **Shell** (like Bash) acts as the command-line interpreter.
* **Essential Commands:**
    * `pwd` & `cd`: Find your location and navigate directories.
    * `mkdir` & `touch`: Create folders and empty files.
    * `ls -lh`: List contents with human-readable file sizes.
    * `cp -r` & `mv`: Copy (recursively) or move/rename files.
    * `rm -r`: Remove files/directories (use with caution!).
* **Process Management:**
    * `ps -ef`: List all system processes.
    * `top` / `htop`: Real-time monitoring of CPU/RAM usage.
    * `kill -9 [PID]`: Forcefully terminate a specific process ID.
    * `fg` & `bg`: Move jobs between the foreground and background.

---

## IV. Integrated Toolchain Summary
| Stage | Primary Tools |
| :--- | :--- |
| **Version Control** | [cite_start]Git, GitHub, Bitbucket [cite: 79] |
| **CI/CD** | [cite_start]Jenkins, GitLab CI, GitHub Actions [cite: 79] |
| **Containers** | [cite_start]Docker, Kubernetes (Orchestration) [cite: 79] |
| **IaC / Config** | [cite_start]Terraform, Ansible, Chef [cite: 79] |
| **Monitoring** | [cite_start]Prometheus, Grafana, Datadog [cite: 79] |