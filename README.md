# KYPO Cyber Range Installation Walkthrough

KYPO Cyber Range Installation Walkthrough developed during an internship at the Department of Information and Communications Engineering (DEIC), Universitat Autònoma de Barcelona (UAB), as part of a research project on cyberrange environments for cybersecurity education.

## Project Overview

The aim for this is to do a (PoC) proof of concept for KYPO Cyberrange in a virtual machine to see all functionalities of KYPO and decide if it's a suitable option for the University Cybersecurity Degree.

## System Specifications

### Host Machine
- **OS**: Windows PC
- **Processor**: Intel i7
- **RAM**: 16GB

### Virtual Environment (VirtualBox)
- **System**: Ubuntu Server
- **Assigned RAM**: 8192 MB (8GB)
- **Assigned CPU**: 6 cores
- **Disk**: 50GB SSD dynamic
- **LVM**: Disabled
- **OpenSSH Server**: Installed
- **Additional snaps**: None
- **User credentials**: `kypo` / `kypo`

## Initial Issues and Solutions

### VirtualBox Hyper-V Conflict
**Problem**: VirtualBox showing the green turtle indicating a conflict with Hyper-V.

**Attempted Solution**: 
```bash
bcdedit /set hypervisorlaunchtype off
```
*This didn't work.*

**Working Solution**: 
1. Open Windows Defender
2. Go to Device Security → Core Isolation Details
3. Turn Memory Integrity **OFF**



## Installation Process

### Prerequisites
- Install OpenStack using Kolla Ansible Project
- Set up the deployment environment (2 major sections)

### Network Configuration
We need 2 network adapters for Kolla Ansible:

![Network Adapter Configuration 1](https://github.com/user-attachments/assets/bacc3110-f485-44ab-861e-060f44fcd7f6)

![Network Adapter Configuration 2](https://github.com/user-attachments/assets/2cf1c853-5349-4282-8983-bdfc8407ef4e)

### Kolla Ansible Installation

**Reference**: [Kolla Ansible QuickStart Guide](https://docs.openstack.org/kolla-ansible/latest/user/quickstart.html)

#### Step-by-Step Installation

1. **Update system packages**:
   ```bash
   sudo apt update
   ```

2. **Install dependencies**:
   ```bash
   sudo apt install git python3-dev libffi-dev gcc libssl-dev libdbus-glib-1-dev
   ```
   > **Note**: Had to use `sudo aptitude install` and respond with "n" then "y"

3. **Install Python virtual environment**:
   ```bash
   sudo apt install python3-venv  # Also done with aptitude
   ```

4. **Create virtual environment**:
   ```bash
   python3 -m venv ~/kolla-ansible/venv
   ```

5. **Activate virtual environment** (use this every time you exit the environment):
   ```bash
   source ~/kolla-ansible/venv/bin/activate
   ```

6. **Upgrade pip**:
   ```bash
   pip install -U pip
   ```

7. **Install Kolla Ansible**:
   ```bash
   pip install git+https://opendev.org/openstack/kolla-ansible@master
   ```

8. **Create Kolla directory**:
   ```bash
   sudo mkdir -p /etc/kolla
   sudo chown $(whoami):$(whoami) /etc/kolla
   ```

9. **Copy configuration files**:
   ```bash
   sudo cp -r ~/kolla-ansible/venv/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
   ```

10. **Copy inventory file**:
    ```bash
    cp ~/kolla-ansible/venv/share/kolla-ansible/ansible/inventory/all-in-one .
    ```

11. **Install dependencies**:
    ```bash
    kolla-ansible install-deps
    ```

12. **Generate passwords**:
    ```bash
    kolla-genpwd
    ```

13. **Configure globals.yml**:
    ```bash
    nano /etc/kolla/globals.yml
    ```
    Add/modify the following:
    ```yaml
    kolla_base_distro: "ubuntu"
    openstack_tag_suffix: "-aarch64"
    network_interface: "eth0"
    neutron_external_interface: "eth1"
    kolla_internal_vip_address: "10.0.2.100"  # Should be in same subnet as eth0
    ```

### Deployment Process

14. **Bootstrap servers**:
    ```bash
    kolla-ansible bootstrap-servers -i ./all-in-one
    ```

15. **Run prechecks**:
    ```bash
    kolla-ansible prechecks -i ./all-in-one
    ```

#### Troubleshooting Issues

**Docker Error**:
```bash
pip install docker
```

**DBus Error**:
```bash
pip3 install dbus-python
```

**Sudo Error**: 
```bash
sudo visudo
# Add: kypo ALL=(ALL) NOPASSWD: ALL
```

**Network Interface Name Conflict**:
```bash
sudo nano /etc/default/grub
# Add: GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
sudo update-grub
sudo reboot
```

**VIP Address Network Issue**:
- Had a misconfiguration on `kolla_internal_vip_address` - ensure it's in the same subnet as eth0

16. **Deploy OpenStack**:
    ```bash
    kolla-ansible deploy -i ./all-in-one
    ```

**Versioning Error with quay.io**:
```bash
nano /etc/kolla/globals.yml
# Comment out: # openstack_tag_suffix: "-aarch64"
# Add: kolla_tag: "master-ubuntu-jammy"
```

Then execute:
```bash
kolla-ansible deploy -i ./all-in-one
```

### Post-Deployment Configuration

17. **Install OpenStack client**:
    ```bash
    pip install python-openstackclient -c https://releases.openstack.org/constraints/upper/master
    ```

18. **Post-deployment setup**:
    ```bash
    kolla-ansible -i ./all-in-one post-deploy
    ```

19. **Configure OpenStack client**:
    ```bash
    mkdir -p ~/.config/openstack
    cp /etc/kolla/clouds.yaml ~/.config/openstack/
    export OS_CLIENT_CONFIG_FILE=/etc/kolla/clouds.yaml
    ```

20. **Verify OpenStack installation**:
    ```bash
    openstack --os-cloud kolla-admin project list
    ```
    ![OpenStack Project List](https://github.com/user-attachments/assets/33861ac5-4b71-4043-91c2-52114b125ffc)

    ```bash
    openstack --os-cloud kolla-admin service list
    ```
    ![OpenStack Service List](https://github.com/user-attachments/assets/3f12f36c-ea15-4a71-8b28-80ad02e7eb95)

**SUCCESS**: Now we have a fully working OpenStack Kolla Ansible with KYPO requirements!


## Application Credentials Setup

### Environment Configuration

1. **Source admin credentials**:
   ```bash
   source /etc/kolla/admin-openrc.sh
   env | grep OS_
   ```
   ![Environment Variables](https://github.com/user-attachments/assets/7545fbf1-6e29-4471-9dc1-97471838f4b4)

2. **Create application credential**:
   ```bash
   openstack application credential create kypo
   ```
   ![Application Credential Creation](https://github.com/user-attachments/assets/5072e657-534e-423d-8557-6d28a97bdb31)

### OpenStack Base Resources Deployment

3. **Create application credentials directory**:
   ```bash
   cd
   mkdir -p app-creds
   ```

4. **Create application credential script**:
   ```bash
   sudo nano app-cred-kypo-openrc.sh
   ```
   Add the following content:
   ```bash
   export OS_AUTH_TYPE=v3applicationcredential
   export OS_AUTH_URL=http://10.0.2.100:5000/v3
   export OS_APPLICATION_CREDENTIAL_ID=xxxxx
   export OS_APPLICATION_CREDENTIAL_SECRET="xxxxx"
   ```

5. **Set permissions**:
   ```bash
   sudo chmod 600 app-cred-kypo-openrc.sh
   sudo chmod 777 app-cred-kypo-openrc.sh
   ```

6. **Source the credentials**:
   ```bash
   source app-cred-kypo-openrc.sh
   cd /home/kypo/kypo-crp-tf-deployment/tf-openstack-base
   ```

## Deployment Environment Preparation

### Install Dependencies

1. **Install Terraform**:
   ```bash
   snap install terraform --classic
   ```

2. **Clone KYPO repository**:
   ```bash
   git clone https://gitlab.ics.muni.cz/muni-kypo-crp/devops/kypo-crp-tf-deployment.git
   ```

3. **Navigate and source credentials**:
   ```bash
   cd kypo-crp-tf-deployment
   source /home/kypo/app-creds/app-cred-kypo-openrc.sh
   cd tf-openstack-base
   ```

### Network Setup

4. **Check external networks**:
   ```bash
   openstack network list --external --column Name
   ```
   **Issue**: Nothing appears initially.

**Solution**:
```bash
sudo apt install isc-dhcp-client
sudo dhclient eth1
ip addr show eth1
# Result: 192.168.56.101/24
```

**Host Machine Network Configuration**:
```
Ethernet adapter Ethernet 2:
   Connection-specific DNS Suffix  . :
   IPv4 Address. . . . . . . . . . . : 192.168.56.1
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . :
```

5. **Create external network**:
   ```bash
   openstack network create external_network --external --provider-network-type flat --provider-physical-network physnet1
   
   openstack subnet create external_subnet \
     --network external_network \
     --subnet-range 192.168.56.0/24 \
     --gateway 192.168.56.1 \
     --allocation-pool start=192.168.56.100,end=192.168.56.200 \
     --dns-nameserver 8.8.8.8 \
     --no-dhcp
   ```

6. **Verify network creation**:
   ```bash
   openstack network list
   openstack subnet list
   openstack network list --external --column Name
   ```
   ![Network List Verification](https://github.com/user-attachments/assets/166bbe9e-cde0-446e-9778-7f87e8ee3238)

### Terraform Configuration

7. **Configure Terraform variables**:
   ```bash
   cp tfvars/deployment.tfvars-template tfvars/deployment.tfvars
   nano tfvars/deployment.tfvars
   ```
   Add:
   ```
   external_network_name = "external_network"
   ```

8. **Initialize Terraform**:
   ```bash
   terraform init
   ```
   ![Terraform Initialization](https://github.com/user-attachments/assets/29268f63-584b-4061-b715-67c32a2187a8)

### Final Deployment

For private OpenStack cloud deployments with admin application credentials (our case), Terraform can deploy all required OpenStack resources.

9. **Deploy all OpenStack resources**:
   ```bash
   terraform apply -var-file tfvars/deployment.tfvars -var-file tfvars/vars-all.tfvars
   ```
   > **Note**: This command will perform extensive operations.

**Issue Encountered**: The command failed due to insufficient HDD space on the machine.

### Recovery Steps

**After reboot**:
```bash
source ~/kolla-ansible/venv/bin/activate
source ~/app-creds/app-cred-kypo-openrc.sh
```
---

## Summary

This documentation provides a complete walkthrough for setting up KYPO Cyber Range using OpenStack and Kolla Ansible. The installation includes proper network configuration, credential management, and deployment of all necessary resources for a functional cybersecurity education environment.
This documentation provides a complete walkthrough for setting up KYPO Cyber Range using OpenStack and Kolla Ansible. The installation includes proper network configuration, credential management, and deployment of all necessary resources for a functional cybersecurity education environment.

The installation failed because my hardware does not meet the minimum requirements. However, I was very close to completing the process. The guides I followed were:  
- [General Deployment Guide](https://gitlab.ics.muni.cz/muni-kypo-crp/devops/kypo-crp-tf-deployment#preparing-the-deployment-environment) **COMPLETED**
- [Deployment of OpenStack Base Resources](https://gitlab.ics.muni.cz/muni-kypo-crp/devops/kypo-crp-tf-deployment/-/blob/master/BASE.md) **FAILED ON LAST STEP**
- [Deployment of KYPO-CRP Helm Application](https://gitlab.ics.muni.cz/muni-kypo-crp/devops/kypo-crp-tf-deployment/-/blob/master/HELM.md) 

> **Note**: Exists a [KYPO Lite](https://gitlab.ics.muni.cz/muni-kypo-crp/devops/kypo-lite) version to make the deployment as simple as possible for testing and evaluation of the platform, but again I don't meet the minimum hardware requeriments.








