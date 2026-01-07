# Class Revision Notes: CI/CD with Kubernetes

*Comprehensive revision notes based on class content — Continuous Integration and Continuous Deployment with Kubernetes.*

---

## Introduction

This class covered **deploying applications into a Kubernetes cluster**, with a focus on **Continuous Integration (CI)** and **Continuous Deployment (CD)** — critical components in modern **DevOps** environments.

---

## Continuous Integration (CI) and Continuous Deployment (CD)

### Continuous Integration (CI)

**CI** automates the integration of code changes from **multiple contributors** into a **single project**.

**Key objectives:**

- **Frequent** code merges  
- **Automated testing**  
- Keeping the **mainline codebase** in a **deployable state**  

### Continuous Deployment (CD)

**CD** extends automation to **deploying changes to production**.

It involves:

- **Infrastructure configuration**  
- **Deployment mechanisms**  
- **Updates to applications without downtime**  

---

## Setting Up a Kubernetes Environment

### Creating a Virtual Machine

- You need **at least one node** in a Kubernetes cluster.  
- Use tools such as **VirtualBox** and **Vagrant** to create the environment.  
- **Vagrant scripts** help **automate** the creation of these virtual environments.  
- Vagrant lets you use **copy-paste scripts** to create VMs with the right configurations.  

### Deploying on Kubernetes

- **Kubernetes** should be **pre-installed** on your VM.  
- Using an existing **Amazon Machine Image (AMI)** with pre-installed Kubernetes is a practical shortcut.  

---

## GitHub Actions and Custom Runners

### GitHub Actions

- **Automates** CI/CD workflows **inside your repository**.  
- Lets you **build, test, package, release, and deploy** code directly on GitHub.  

### Custom Runners (Self-Hosted)

- Instead of **GitHub-hosted** runners, you can use **self-hosted** runners.  
- **Configure your own machine** (Linux recommended) as a runner.  
- **Register** the runner in GitHub and use it in your workflows.  

**Configuration:**

- Use **config.sh** to register the runner.  
- Set up the runner with the right **labels** and **security** settings.  

---

## Project Demonstration and Teamwork

- Each **team** works on a **project** and **presents** it.  
- The project involves **implementing CI/CD** for an application using **GitHub Actions** (and other tools).  
- Although work is **collaborative**, **each person** is responsible for their **own contributions**, especially when **asked during presentations**.  

---

## Testing and Quality Assurance

- Use a **structured testing** approach with environments such as:  
  - **System Integration Testing (SIT)**  
  - **Performance Testing**  
  - **Security Testing**  
- Testing phases are often **automated** to:  
  - Reduce **human error**  
  - Improve **test coverage**  

---

## Practical Considerations

- **CI and CD** are usually **separate processes** in well-designed setups, to keep a clear **separation of concerns**.  
- **Continuous deployment pipelines** may need **specific configurations** depending on:  
  - Your **application’s requirements**  
  - Your **hosting infrastructure**  

---

## Summary

| Topic | Takeaway |
|-------|----------|
| **CI** | Frequent merges, automated tests, mainline always deployable |
| **CD** | Automated deploy to production, zero-downtime updates |
| **Kubernetes setup** | VM (e.g. Vagrant + VirtualBox), pre-installed K8s or AMI |
| **GitHub Actions** | Build, test, package, release, deploy from the repo |
| **Custom runners** | Self-hosted runner (e.g. Linux), config.sh, labels, security |
| **Testing** | SIT, performance, security; automate where possible |
| **CI vs CD** | Keep them separate; tune pipelines to app and infrastructure |

*Use your own notes and recorded sessions for more detail and context.*