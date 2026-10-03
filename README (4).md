# 🚀 Enterprise Multi-OS Docker Deployment with Ansible on AWS

> **Production-style Ansible automation project demonstrating AWS
> Dynamic Inventory, reusable roles, multi-OS configuration management,
> idempotent deployments, and automated validation.**

[![Ansible](https://img.shields.io/badge/Ansible-2.21-red?logo=ansible)](https://www.ansible.com/)
[![AWS](https://img.shields.io/badge/AWS-EC2-orange?logo=amazon-aws)](https://aws.amazon.com/ec2/)
[![Docker](https://img.shields.io/badge/Docker-29.8.2-2496ED?logo=docker)](https://www.docker.com/)
[![Linux](https://img.shields.io/badge/Linux-Multi--OS-FCC624?logo=linux)](https://www.linux.org/)
[![IaC](https://img.shields.io/badge/Infrastructure%20as%20Code-Ansible-black?logo=ansible)](https://www.ansible.com/)

------------------------------------------------------------------------

## 📌 Project Overview

This project automates the **installation, configuration, and validation
of Docker Engine across heterogeneous AWS EC2 environments** using a
reusable Ansible role.

Instead of maintaining a static inventory or writing separate playbooks
for every operating system, the project uses:

-   **AWS EC2 Dynamic Inventory** for automatic host discovery
-   **AWS tags** for environment and role-based grouping
-   **Reusable Ansible roles**
-   **OS-aware task execution**
-   **Official Docker repositories**
-   **Privilege escalation**
-   **Idempotent configuration**
-   **Automated service and version validation**
-   **Dependency management through `requirements.yml`**

The environment was validated across **four AWS EC2 instances**:

  Platform    Role
  ----------- -------------
  Ubuntu      Docker host
  Ubuntu      Docker host
  Debian      Docker host
  RHEL 10.2   Docker host

All four hosts were successfully configured and validated with **Docker
Engine 29.8.2**.

------------------------------------------------------------------------

# 🏗️ Architecture

``` text
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
```

------------------------------------------------------------------------

# 🎯 Business / Engineering Objective

The goal is to create a **reusable and scalable automation solution**
for organizations operating Linux workloads across multiple
distributions.

### Problem

Manually installing Docker across different Linux distributions creates:

-   Configuration drift
-   Repetitive operational work
-   OS-specific installation complexity
-   Inconsistent versions/configuration
-   Higher maintenance overhead
-   Poor scalability

### Solution

Ansible provides a declarative automation layer that:

1.  Discovers EC2 instances dynamically.
2.  Groups hosts using AWS metadata/tags.
3.  Detects the operating-system family.
4.  Executes the appropriate installation workflow.
5.  Installs Docker and supporting plugins.
6.  Enables and starts the Docker service.
7.  Validates the deployment automatically.

------------------------------------------------------------------------

# 🛠️ Technology Stack

  Technology              Purpose
  ----------------------- ---------------------------------------
  AWS EC2                 Compute infrastructure
  AWS Dynamic Inventory   Automatic host discovery
  Ansible                 Configuration management / automation
  Ansible Roles           Reusable automation architecture
  Ubuntu                  Debian-family target
  Debian                  Debian-family target
  RHEL 10.2               RedHat-family target
  Docker CE               Container runtime
  Docker Compose Plugin   Container orchestration
  Docker Buildx           Image building
  Python / boto3          AWS inventory integration
  Git / GitHub            Source control
  Ansible Galaxy          Role distribution

------------------------------------------------------------------------

# 📁 Project Structure

``` text
Day-05/
└── docker-role-demo/
    │
    ├── inventory/
    │   └── aws_ec2.yml
    │
    ├── docker/
    │   ├── defaults/
    │   │   └── main.yml
    │   │
    │   ├── handlers/
    │   │   └── main.yml
    │   │
    │   ├── meta/
    │   │   └── main.yml
    │   │
    │   ├── tasks/
    │   │   ├── main.yml
    │   │   ├── debian.yml
    │   │   └── redhat.yml
    │   │
    │   ├── templates/
    │   ├── vars/
    │   │   └── main.yml
    │   │
    │   └── README.md
    │
    ├── requirements.yml
    ├── site.yml
    ├── README.md
    └── .gitignore
```

------------------------------------------------------------------------

# ☁️ AWS Dynamic Inventory

A static inventory such as:

``` ini
[ubuntu]
10.x.x.x

[debian]
10.x.x.x

[rhel]
10.x.x.x
```

does not scale well in dynamic cloud environments.

This project uses the `amazon.aws.aws_ec2` inventory plugin.

Example:

``` yaml
---
plugin: amazon.aws.aws_ec2

regions:
  - ap-southeast-2

filters:
  instance-state-name:
    - running

  tag:Application:
    - ansible-day05

keyed_groups:
  - key: ec2_tags.Environment
    prefix: env
    separator: "_"

  - key: ec2_tags.Application
    prefix: app
    separator: "_"

  - key: ec2_tags.Role
    prefix: role
    separator: "_"

  - key: ec2_tags.OS
    prefix: os
    separator: "_"

  - key: ec2_tags.Name
    prefix: name
    separator: "_"

compose:
  ansible_host: public_ip_address
```

### Why dynamic inventory?

New EC2 instances can be launched and automatically discovered based on
AWS metadata and tags without manually editing an inventory file.

This is particularly useful in:

-   Auto Scaling environments
-   Ephemeral infrastructure
-   Dev/Test environments
-   Multi-account AWS environments
-   Large EC2 fleets

------------------------------------------------------------------------

# 🏷️ AWS Tag-Based Host Grouping

The inventory uses AWS tags to build Ansible groups.

Example:

``` text
Application = ansible-day05
Role        = docker
Environment = dev
OS          = rhel
```

This allows targeting:

``` bash
ansible role_docker -i inventory/aws_ec2.yml -m ping
```

instead of hard-coding individual IP addresses.

------------------------------------------------------------------------

# 🔧 Ansible Role Design

The role separates common logic from OS-specific implementation.

## `tasks/main.yml`

``` yaml
---
- name: Install Docker on Debian family
  ansible.builtin.include_tasks: debian.yml
  when: ansible_facts.os_family == "Debian"

- name: Install Docker on RedHat family
  ansible.builtin.include_tasks: redhat.yml
  when: ansible_facts.os_family == "RedHat"
```

This provides a clean dispatcher pattern.

------------------------------------------------------------------------

# 🐧 Debian / Ubuntu Workflow

The Debian-family workflow:

1.  Installs required packages.
2.  Creates the Docker keyring directory.
3.  Downloads Docker's GPG key.
4.  Configures the official Docker repository.
5.  Installs Docker Engine.
6.  Installs Buildx.
7.  Installs Docker Compose plugin.
8.  Enables Docker.
9.  Starts Docker.

Packages:

``` text
docker-ce
docker-ce-cli
containerd.io
docker-buildx-plugin
docker-compose-plugin
```

------------------------------------------------------------------------

# 🔴 RHEL Workflow

The RHEL workflow:

1.  Removes obsolete Docker repository configuration.
2.  Adds the official Docker CE repository.
3.  Refreshes package metadata.
4.  Installs Docker Engine and plugins.
5.  Enables Docker.
6.  Starts Docker.

Packages:

``` text
docker-ce
docker-ce-cli
containerd.io
docker-buildx-plugin
docker-compose-plugin
```

The project was tested against:

``` text
Red Hat Enterprise Linux 10.2
x86_64
```

------------------------------------------------------------------------

# 🔐 Privilege Escalation

Docker installation and service management require elevated privileges.

The playbook therefore uses:

``` yaml
become: true
```

Example:

``` yaml
- name: Deploy and configure Docker
  hosts: role_docker
  become: true
  gather_facts: true

  roles:
    - role: ../docker
```

For ad-hoc Docker validation:

``` bash
ansible all \
  -i inventory/aws_ec2.yml \
  -b \
  -m command \
  -a "docker info"
```

The `-b` option ensures the command has access to the Docker Unix socket
where required.

------------------------------------------------------------------------

# ♻️ Idempotency

Idempotency is a core Ansible design principle.

The role is designed so that repeated executions converge the hosts
toward the desired state rather than reinstalling or unnecessarily
modifying Docker.

Example:

``` bash
ansible-playbook \
  -i inventory/aws_ec2.yml \
  site.yml
```

A subsequent execution should result in minimal or zero unnecessary
changes.

This allows the role to safely participate in:

-   CI/CD pipelines
-   Server provisioning
-   Disaster recovery
-   Environment rebuilds
-   Configuration drift remediation

------------------------------------------------------------------------

# ✅ Automated Validation

The playbook validates Docker after installation.

Example:

``` yaml
- name: Verify Docker service
  ansible.builtin.service_facts:

- name: Validate Docker service is running
  ansible.builtin.assert:
    that:
      - ansible_facts.services['docker.service'].state == 'running'
    fail_msg: "Docker service is not running"
    success_msg: "Docker service is running"
```

Docker version validation:

``` yaml
- name: Check Docker version
  ansible.builtin.command:
    cmd: docker --version
  changed_when: false
  register: docker_version

- name: Display Docker version
  ansible.builtin.debug:
    msg: "{{ docker_version.stdout }}"
```

------------------------------------------------------------------------

# 🧪 Verification Commands

### Validate dynamic inventory

``` bash
ansible-inventory \
  -i inventory/aws_ec2.yml \
  --graph
```

### Test connectivity

``` bash
ansible all \
  -i inventory/aws_ec2.yml \
  -m ping
```

### Validate Docker version

``` bash
ansible role_docker \
  -i inventory/aws_ec2.yml \
  -b \
  -m command \
  -a "docker --version"
```

### Validate Docker service

``` bash
ansible role_docker \
  -i inventory/aws_ec2.yml \
  -b \
  -m command \
  -a "systemctl is-active docker"
```

### Validate Docker daemon

``` bash
ansible role_docker \
  -i inventory/aws_ec2.yml \
  -b \
  -m command \
  -a "docker info"
```

### Syntax check

``` bash
ansible-playbook \
  -i inventory/aws_ec2.yml \
  site.yml \
  --syntax-check
```

### Check mode

``` bash
ansible-playbook \
  -i inventory/aws_ec2.yml \
  site.yml \
  --check
```

------------------------------------------------------------------------

# 📦 Dependencies

`requirements.yml`:

``` yaml
---
collections:
  - name: amazon.aws
    version: ">=10.0.0"
```

Install dependencies:

``` bash
ansible-galaxy collection install -r requirements.yml
```

Python dependencies:

``` bash
pip install boto3 botocore
```

------------------------------------------------------------------------

# 🔒 Security Practices

The project follows several infrastructure security practices:

-   AWS private keys are excluded through `.gitignore`.
-   Secrets are not stored in source code.
-   Ansible Vault can be used for sensitive variables.
-   Privilege escalation is explicitly controlled.
-   Official Docker repositories are used.
-   Host discovery is based on AWS metadata rather than manually
    maintained IP lists.
-   Docker access is validated after installation.

Example `.gitignore`:

``` gitignore
.venv/
*.pem
*.key
.env
.env.*
*.retry
*.log
.vscode/
.idea/
```

> **Never commit AWS private keys, API tokens, passwords, or other
> credentials to GitHub.**

------------------------------------------------------------------------

# 📊 Deployment Result

The automation was validated across four AWS EC2 instances:

``` text
┌─────────────────────────────────────────────┐
│          Multi-OS Docker Deployment         │
├──────────────────────┬──────────────────────┤
│ Host OS              │ Result               │
├──────────────────────┼──────────────────────┤
│ Ubuntu               │ ✅ Docker 29.8.2     │
│ Ubuntu               │ ✅ Docker 29.8.2     │
│ Debian               │ ✅ Docker 29.8.2     │
│ RHEL 10.2            │ ✅ Docker 29.8.2     │
└──────────────────────┴──────────────────────┘
```

Final Ansible deployment status:

``` text
failed=0
unreachable=0
```

Docker service validation:

``` text
active
active
active
active
```

------------------------------------------------------------------------

# 🧠 Engineering Challenges Solved

## 1. Static inventory → Dynamic inventory

Instead of maintaining EC2 IP addresses manually, AWS tags drive host
discovery and grouping.

## 2. Multi-OS package management

Debian-family and RedHat-family systems use different package managers
and repository configuration mechanisms.

The role handles this through:

``` text
Debian → apt
RHEL   → dnf
```

## 3. RHEL Docker repository issue

The RHEL environment initially contained an invalid Docker repository
configuration.

The issue was diagnosed using:

``` bash
dnf repolist
file /etc/yum.repos.d/docker-ce.repo
```

The repository was found to contain HTML rather than a valid repository
definition.

The invalid configuration was removed and the official Docker repository
was restored.

## 4. RHEL kernel / Docker networking startup

Docker packages installed successfully, but the daemon initially failed
while configuring bridge/NAT networking.

The system had installed a newer kernel and required a reboot.

After rebooting into:

``` text
6.12.0-211.61.1.el10_2.x86_64
```

Docker successfully initialized.

## 5. Docker socket permissions

Ad-hoc commands executed as `ec2-user` initially returned:

``` text
permission denied while trying to connect to the Docker API
```

The issue was resolved for Ansible validation using privilege
escalation:

``` bash
ansible ... -b -m command -a "docker info"
```

This demonstrates understanding of Linux permissions, Unix sockets, and
Ansible privilege escalation.

------------------------------------------------------------------------

# 💼 Why This Project Matters

This project demonstrates more than simply installing Docker.

It demonstrates practical DevOps engineering concepts:

### Infrastructure

-   AWS EC2
-   Cloud metadata
-   Dynamic inventory
-   Instance tagging

### Configuration Management

-   Ansible
-   Roles
-   Idempotency
-   Conditional task execution
-   Privilege escalation

### Linux

-   systemd
-   package management
-   repository configuration
-   kernel modules
-   service troubleshooting
-   Unix socket permissions

### Containers

-   Docker Engine
-   containerd
-   Buildx
-   Docker Compose
-   Docker networking

### Engineering Practices

-   Reusable automation
-   Dependency management
-   Validation
-   Troubleshooting
-   Git-based version control
-   Galaxy-ready role architecture

------------------------------------------------------------------------

# 🎤 Interview Discussion

A recruiter or interviewer can use this project to explore several
real-world topics.

### Q: Why use dynamic inventory?

**Answer:**

> In AWS environments, EC2 instances are often ephemeral and their IP
> addresses can change. Dynamic inventory allows Ansible to discover
> instances directly from AWS and organize them using metadata and tags,
> reducing manual inventory maintenance.

### Q: How did you support multiple operating systems?

**Answer:**

> I used a common role entry point and dispatched to OS-specific task
> files based on `ansible_facts.os_family`. Debian-family systems use
> APT while RedHat-family systems use DNF.

### Q: How did you make the role idempotent?

**Answer:**

> I used Ansible modules such as `package`, `file`, `get_url`,
> `apt_repository`, and `systemd` with explicit desired states. This
> allows repeated executions to converge the system without unnecessary
> changes.

### Q: How did you troubleshoot the RHEL Docker failure?

**Answer:**

> I separated the problem into repository, package, service, and
> kernel/networking layers. First I verified the repository, then
> package availability, then Docker service logs using `journalctl`. The
> logs showed an iptables/netfilter networking failure. The installation
> had also updated the kernel and requested a reboot. After rebooting
> into the new kernel, Docker initialized successfully.

### Q: Why did `docker info` fail from Ansible initially?

**Answer:**

> The command was executed as the non-root SSH user, which did not have
> permission to access `/var/run/docker.sock`. The playbook already uses
> privilege escalation, so validation was performed with Ansible's `-b`
> option.

------------------------------------------------------------------------

# 🚀 Future Improvements

Potential next iterations:

-   Add Molecule-based role testing
-   Add GitHub Actions CI
-   Add Ansible Lint
-   Add automated syntax validation
-   Add automated multi-OS testing
-   Parameterize Docker package versions
-   Add configurable Docker daemon settings
-   Add Docker daemon hardening
-   Add configurable storage drivers
-   Add configurable Docker users
-   Add rollback/uninstall tasks
-   Publish the role to Ansible Galaxy
-   Integrate the role into a CI/CD pipeline
-   Add AWS Systems Manager integration
-   Add CloudWatch monitoring
-   Add security scanning

------------------------------------------------------------------------

# 🌐 Publishing Strategy

The role is designed to be published as a reusable Ansible Galaxy role.

Recommended workflow:

``` text
Developer
   │
   ▼
Git Repository
   │
   ▼
GitHub
   │
   ▼
Ansible Galaxy
   │
   ▼
Reusable Ansible Role
   │
   ├── Development
   ├── QA
   ├── Staging
   └── Production
```

Example consumer workflow:

``` yaml
---
- name: Configure Docker hosts
  hosts: role_docker
  become: true

  roles:
    - docker
```

------------------------------------------------------------------------

# 📈 Project Outcomes

### Before automation

``` text
Manual EC2 selection
       ↓
Manual SSH
       ↓
OS detection
       ↓
Manual repository setup
       ↓
Manual Docker installation
       ↓
Manual service configuration
       ↓
Manual validation
```

### After automation

``` text
AWS Tags
   ↓
Dynamic Inventory
   ↓
Ansible Role
   ↓
OS Detection
   ↓
Repository Configuration
   ↓
Docker Installation
   ↓
Service Management
   ↓
Automated Validation
```

The result is a repeatable workflow that can be applied consistently
across heterogeneous AWS Linux environments.

------------------------------------------------------------------------

# 🏆 Key Skills Demonstrated

``` text
AWS
├── EC2
├── Instance Tags
└── Dynamic Inventory

Ansible
├── Roles
├── Dynamic Inventory
├── Facts
├── Conditionals
├── Privilege Escalation
├── Idempotency
├── Validation
└── Dependency Management

Linux
├── systemd
├── APT
├── DNF
├── Repository Management
├── Kernel Troubleshooting
├── iptables/netfilter
└── Unix Socket Permissions

Docker
├── Docker Engine
├── containerd
├── Buildx
├── Compose
└── Networking

DevOps
├── Infrastructure Automation
├── Configuration Management
├── Git
├── CI/CD readiness
└── Reusable Infrastructure
```

------------------------------------------------------------------------

# 👨‍💻 Author

**Rajesh Kumar Samal**

AWS / DevOps Engineer

Focus Areas:

-   AWS
-   DevOps
-   Ansible
-   Docker
-   Linux
-   CI/CD
-   Infrastructure Automation
-   Cloud Automation

------------------------------------------------------------------------

## ⭐ If you found this project useful

Feel free to fork the repository, experiment with the role, and extend
it with Molecule testing, CI/CD, monitoring, and security hardening.

> **This project is intentionally designed as a reusable automation
> component rather than a one-time server setup script.**
