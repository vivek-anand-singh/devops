# Revision Notes: Terraform and Infrastructure as Code

---

## Introduction to Terraform

**Terraform** is an open-source tool that lets you **manage cloud infrastructure through code**, making it easier to **automate and control** IT resources across multiple providers (e.g. AWS, Azure, Google Cloud).

### Key Characteristics

- **Platform agnostic** — Works with many cloud providers, so you can manage infrastructure across different platforms.
- **Infrastructure as Code (IaC)** — Manages infrastructure using code: more **consistency**, fewer errors from manual configuration.

---

## Lifecycle Commands in Terraform

Terraform’s main workflow uses **four commands**:

### 1. Terraform Init

- **Initializes** a working directory that contains Terraform configuration files.
- **Downloads** required **modules** and **providers** referenced in the config.
- **Locks** provider versions for compatibility.

### 2. Terraform Plan

- **Compares** current infrastructure state with the **desired state** in your config and produces an **execution plan**.
- Lets you **preview** changes **without applying** them, reducing risk of unintended changes.

### 3. Terraform Apply

- **Applies** the changes needed to match the desired state from your configuration.
- Updates infrastructure in a **controlled** way based on the plan.

### 4. Terraform Destroy

- **Removes** all resources defined in your Terraform configuration.
- Used to **clean up** infrastructure.

---

## Advantages of Using Terraform

| Advantage | Description |
|-----------|-------------|
| **Speed and efficiency** | Automates creation and updates; much faster and cheaper than manual provisioning. |
| **Consistency** | Same config produces same result; fewer discrepancies than manual setup. |
| **Version control** | Config can live in Git (or similar) for **rollback**, **audit**, and **collaboration**. |

---

## Supporting Tools and Concepts

### Configuration Management Tools

Tools like **Ansible**, **Puppet**, and **Chef** work **alongside** Terraform to manage **software configuration** and **updates** on the infrastructure Terraform provisions.

### Server Templating Tools

- **Vagrant** — Create and manage virtual environments.
- **Docker** — Create containerized services.

Both support the infrastructure lifecycle that Terraform manages.

---

## Practical Demonstration

### Example Use Case: Creating an EC2 Instance

1. **Define** infrastructure in **`.tf`** files (e.g. EC2 instance and related resources).
2. Run Terraform commands:
   - **init** — set up working directory and providers.
   - **plan** — review what will be created/changed/destroyed.
   - **apply** — create or update resources to match the config.
3. Use **Terraform state** to track resources and manage changes and deletions.

---

## Summary Table

| Command | Purpose |
|---------|---------|
| `terraform init` | Initialize directory; download providers/modules; lock versions |
| `terraform plan` | Show execution plan (no changes applied) |
| `terraform apply` | Apply changes to reach desired state |
| `terraform destroy` | Remove all managed resources |

---

## Conclusion

Terraform is a core tool for **modern infrastructure management** and **IaC**. It provides:

- **Speed** and **consistency**
- A clear **audit trail** via version control  
- Strong support for **DevOps** practices  

Refer to the **official Terraform documentation** for advanced use cases and commands not covered in class.