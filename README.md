import pypandoc
from pathlib import Path

md = r'''# 🚀 Enterprise Multi-OS Docker Deployment with Ansible on AWS

> **Production-style Ansible automation project demonstrating AWS Dynamic Inventory, reusable roles, multi-OS configuration management, idempotent deployments, and automated validation.**

[![Ansible](https://img.shields.io/badge/Ansible-2.21-red?logo=ansible)](https://www.ansible.com/)
[![AWS](https://img.shields.io/badge/AWS-EC2-orange?logo=amazon-aws)](https://aws.amazon.com/ec2/)
[![Docker](https://img.shields.io/badge/Docker-29.8.2-2496ED?logo=docker)](https://www.docker.com/)
[![Linux](https://img.shields.io/badge/Linux-Multi--OS-FCC624?logo=linux)](https://www.linux.org/)
[![IaC](https://img.shields.io/badge/Infrastructure%20as%20Code-Ansible-black?logo=ansible)](https://www.ansible.com/)

---

## 📌 Project Overview

This project automates the **installation, configuration, and validation of Docker Engine across heterogeneous AWS EC2 environments** using a reusable Ansible role.

Instead of maintaining a static inventory or writing separate playbooks for every operating system, the project uses:

- **AWS EC2 Dynamic Inventory** for automatic host discovery
- **AWS tags** for environment and role-based grouping
- **Reusable Ansible roles**
- **OS-aware task execution**
- **Official Docker repositories**
- **Privilege escalation**
- **Idempotent configuration**
- **Automated service and version validation**
- **Dependency management through `requirements.yml`**

The environment was validated across **four AWS EC2 instances**:

| Platform | Role |
|---|---|
| Ubuntu | Docker host |
| Ubuntu | Docker host |
| Debian | Docker host |
| RHEL 10.2 | Docker host |

All four hosts were successfully configured and validated with **Docker Engine 29.8.2**.

---

# 🏗️ Architecture

```text
                         ┌─────────────────────────┐
                         │       AWS Account       │
                         │     ap-southeast-2      │
                         └────────────┬────────────┘
                                      │
                                      ▼
                         ┌─────────────────────────┐
                         │      EC2 Instances      │
                         │                         │
                         │  Ubuntu × 2             │
                         │  Debian × 1             │
                         │  RHEL 10.2 × 1          │
                         └────────────┬────────────┘
                                      │
                                      │ AWS Tags
                                      ▼
                         ┌─────────────────────────┐
                         │ AWS Dynamic Inventory   │
                         │      aws_ec2.yml        │
                         └────────────┬────────────┘
                                      │
                                      ▼
                         ┌─────────────────────────┐
                         │     Ansible Controller  │
                         │       WSL / Linux      │
                         └────────────┬────────────┘
                                      │
                                      ▼
                         ┌─────────────────────────┐
                         │     role_docker group   │
                         └────────────┬────────────┘
                                      │
                     ┌────────────────┴────────────────┐
                     │                                 │
                     ▼                                 ▼
             Debian Family                       RedHat Family
             debian.yml                          redhat.yml
                     │                                 │
                     ▼                                 ▼
              Docker CE Repo                    Docker CE Repo
                     │                                 │
                     └────────────────┬────────────────┘
                                      ▼
                         ┌─────────────────────────┐
                         │      Docker Engine      │
                         │        29.8.2           │
                         └────────────┬────────────┘
                                      │
                                      ▼
                         ┌─────────────────────────┐
                         │ Automated Validation    │
                         │                         │
                         │ Service Status          │
                         │ Docker Version          │
                         │ docker info             │
                         └─────────────────────────┘
