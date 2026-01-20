# Nested ESXi Deployment with Ansible  
### Fully Unattended ESXi 9.x Install via Kickstart + NFS

---

## Overview

This project provides a **fully unattended Ansible workflow** to deploy **nested ESXi 9.x hosts** on a physical ESXi server.

The deployment uses:

- **Ansible** automation
- **community.vmware** modules
- **Kickstart (ks.cfg)** files hosted on **NFS**
- **Custom ESXi ISO boot parameters**
- Automatic **post-install cleanup**

The end goal is simple:

> **Run one Ansible playbook → get fully configured nested ESXi hosts → no manual interaction**

---

##  Architecture

### Components

- **Ansible Jumpbox** (Ubuntu)
- **Physical ESXi Host**
- **NFS Server**
  - Kickstart files
  - ESXi datastore
- **Nested ESXi Virtual Machines**

### High-Level Flow

1. Ansible renders **per-host Kickstart files** to an NFS share  
2. A **custom ESXi ISO** is generated with boot parameters pointing to NFS  
3. ISO is uploaded to the physical ESXi datastore  
4. Nested ESXi VMs are created and powered on  
5. ESXi installs **fully unattended**  
6. ISO is disconnected and deleted  
7. NFS datastore is mounted on each nested ESXi host  

---

## Prerequisites

### Physical Environment

- Physical ESXi host (tested with ESXi 8.x and 9.x)
- Datastore accessible to ESXi (SSD or NFS)
- NFS server reachable by:
  - Ansible jumpbox
  - Nested ESXi hosts

### Networking

- DHCP available on the management network
- Portgroup available for nested ESXi
- MTU 9000 supported end-to-end (recommended)

---

## Jumpbox Preparation (Ubuntu)

### 1. Update the system

```bash
sudo apt update && sudo apt upgrade -y
2. Install required packages
bash
Copy code
sudo apt install -y \
  python3 \
  python3-venv \
  python3-pip \
  git \
  xorriso \
  nfs-common
3. Create and activate a Python virtual environment
bash
Copy code
python3 -m venv ansible-venv
source ansible-venv/bin/activate
4. Install Ansible
bash
Copy code
pip install --upgrade pip
pip install ansible
5. Install VMware Ansible collection
bash
Copy code
ansible-galaxy collection install community.vmware
Shell Configuration
Disable host key checking
This prevents Ansible SSH prompts from breaking automation.

Add the following to ~/.bashrc:

bash
Copy code
export ANSIBLE_HOST_KEY_CHECKING=False
Apply immediately:

bash
Copy code
source ~/.bashrc
Clone the Repository
bash
Copy code
git clone https://github.com/CybrG33k/nested-esxi-ansible.git
cd nested-esxi-ansible
Configuration
vars/creds.yaml
This file defines:

Physical ESXi credentials

Datastore names

Nested ESXi host definitions

CPU and memory sizing

Networking

Edit this file before running the playbook.

Example
yaml
Copy code
physical_esxi:
  esxi_host: 10.10.10.100
  esxi_username: root
  esxi_password: YOUR_PASSWORD
  datastore_name: SSD

nested_info:
  - name: vcf-mgmt-esxi-1
    staticip: 10.10.10.200
    fqdn: vcf-mgmt-esxi-1.homelab.net
  - name: vcf-mgmt-esxi-2
    staticip: 10.10.10.201
    fqdn: vcf-mgmt-esxi-2.homelab.net

nested_info_global:
  iso_location: /path/to/VMware-VMvisor-Installer.iso
  esxi_cpu: 8
  esxi_mem_mb: 65536
  networks:
    - name: "VM Network"
Kickstart Behavior
The Kickstart template (roles/deployesxi/templates/ks.j2) performs:

Fully unattended ESXi installation

DHCP-based management networking

IPv6 disabled

vSwitch0 MTU set to 9000

vmk0 MTU set to 9000

Promiscuous mode enabled

MAC address changes enabled

Forged transmits enabled

NFS datastore mounted post-install

Removal of temporary install datastore (remote-install-location)

Deployment
Run the full deployment
bash
Copy code
ansible-playbook -i localhost, -c local main.yaml
Kickstart-only run (optional)
bash
Copy code
ansible-playbook -i localhost, -c local main.yaml --tags kickstart
What You Should See
Nested ESXi VMs created on the physical host

ESXi installer runs without prompts

Hosts reboot cleanly after installation

ISO automatically disconnected and deleted

NFS datastore mounted

vSwitch and VMkernel networking ready for nested virtualization

Cleanup Behavior
After installation:

ISO is disconnected from the VM CD-ROM

ISO is removed from the datastore

Temporary files on the jumpbox are deleted

Troubleshooting
ISO cannot be deleted
Ensure the ISO is disconnected from the VM CD-ROM

The cleanup role handles this automatically

Kickstart not detected
Verify NFS is reachable from the nested ESXi host

Confirm BOOT.CFG contains the correct ks=nfs:// reference

Network issues in nested ESXi
Verify MTU consistency (9000)

Ensure the portgroup allows:

Promiscuous mode

MAC address changes

Forged transmits

Credits
Inspired by the original structure from canad1an.

Heavily extended and customized for:

ESXi 9.x

NFS-based Kickstart delivery

Fully unattended installations

Automated cleanup
