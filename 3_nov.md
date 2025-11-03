# Engineering Master Notes: DevOps, AWS, Linux & Shell Scripting

## 1. DevOps Introduction
* [cite_start]**Definition:** A set of practices, cultural philosophies, and tools combining software development (Dev) and IT operations (Ops) to deliver applications faster and more reliably[cite: 4].
* [cite_start]**Key Goals:** Break silos between teams, automate repetitive tasks, and ensure continuous delivery of high-quality software[cite: 6, 7, 8].
* [cite_start]**Lifecycle:** An iterative process consisting of Plan, Code, Build, Test, Release, Deploy, Operate, and Monitor[cite: 55, 56].



* [cite_start]**Principles:** Focuses on collaboration, automation, CI/CD, measurement, and Infrastructure as Code (IaC)[cite: 60, 62, 64, 70].

---

## 2. AWS Fundamentals
* **Regions:** Geographically isolated areas containing multiple Availability Zones (AZs).
* **Availability Zones (AZs):** Physically separate datacenters within a Region connected via low-latency links.
* **VPC & Subnets:**
    * **VPC:** An isolated virtual network within a Region.
    * **Subnets:** Smaller network segments; Public subnets are internet-facing, while Private subnets often use NAT Gateways.
* **EC2 Configuration:**
    * **AMI:** The OS image (Ubuntu, Amazon Linux, etc.).
    * **Instance Type:** Determines CPU/RAM (e.g., t3.micro).
    * **Security Groups:** Virtual firewalls for inbound/outbound traffic.

---

## 3. Linux Fundamentals
* **Architecture:** The **Kernel** manages hardware (CPU, memory, devices), while the **Shell** acts as the command-line interpreter.
* **Basic Commands:**
    * `pwd`: Current directory | `cd`: Change directory | `mkdir`: Create directory.
    * `touch`: Create empty file | `cat`: View file content | `echo`: Print text.
* **File Management:**
    * `ls -l`: Long list | `cp -r`: Recursive copy | `mv`: Move/Rename | `rm`: Remove.
    * `find`: Search for files | `df -h`: Disk space usage.
* **Process Management:**
    * `ps -ef`: List all processes | `top`: Real-time monitoring | `kill -9`: Force terminate.

---

## 4. Shell Scripting Notes
* **Shebang (`#!`):** Tells the system which interpreter to use (e.g., `#!/bin/bash` or `#!/bin/sh`).
* **Bash vs. Sh:**
    * **Bash:** Rich features like arrays, `${var^^}` (uppercase), and `[[ ... ]]`.
    * **Sh:** Basic, POSIX-compliant, and faster but lacks advanced string manipulation.
* **Common Script Examples:**
    * **Variable Reference:** `${str}` retrieves values; `${#str}` gets length.
    * **Case Conversion:** `${str^^}` (Uppercase) and `${str,,}` (Lowercase).
    * **Conditions:** `if [ -f "$file" ]` checks if a file exists.
* **Modular Scripts:** Use `source "$(dirname "$0")/utils.sh"` to import functions from other files.

---

## 5. Advanced Permissions (ACLs)
* **Definition:** Access Control Lists allow fine-grained permissions for specific users/groups on a single file.
* **Commands:**
    * `getfacl`: View permissions | `setfacl -m u:user:rwx`: Modify permissions.
    * `setfacl -x`: Remove specific entry | `setfacl -b`: Remove all ACLs.
* **Mask Field:** Defines the maximum allowed permissions for all users except the owner.

---

## 6. Integrated Toolchain Summary
| Stage | Tools / Examples |
| :--- | :--- |
| **Version Control** | [cite_start]Git, GitHub, GitLab, Bitbucket [cite: 79] |
| **CI/CD** | [cite_start]Jenkins, GitLab CI, GitHub Actions [cite: 79] |
| **Containerization** | [cite_start]Docker, Kubernetes (Orchestration) [cite: 79] |
| **IaC** | [cite_start]Terraform, Ansible, CloudFormation [cite: 79] |
| **Monitoring** | [cite_start]Prometheus, Grafana, ELK Stack [cite: 79] |