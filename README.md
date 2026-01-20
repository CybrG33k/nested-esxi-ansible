# VMware vSphere Nested ESXi Environment - Deployed by automatically installing the ISO and configuring via KS.CFG file (kickstart file)

This playbook will deploy one or many nested ESXi hosts on to your physical ESXi vSphere host. The purpose is to create a nested esxi environment for various purposes. In my case, for deploying a VCF environment. To see more details, please see my blog post: https://mattadam.com/2024/01/09/deploy-nested-esxi-vms-using-a-kickstart-ks-script-and-ansible-automation/

## Variables
/vars/creds.yaml - There's a few variables to set in the creds file for this script to work

```
nested_info: # This is a list of the ESXI VMs that you want to create, add more or remove some as necessary
  - name: vcf-mgmt-esxi-1 # VM name
    staticip: 192.168.3.16 # Static IP address of the VM
    fqdn: vcf-mgmt-esxi-1.home.lab # FQDN, used for setting the hostname, and certificate generation
  - name: vcf-mgmt-esxi-2
    staticip: 192.168.3.17
    fqdn: vcf-mgmt-esxi-2.home.lab
  - name: vcf-mgmt-esxi-3
    staticip: 192.168.3.18
    fqdn: vcf-mgmt-esxi-3.home.lab
  - name: vcf-mgmt-esxi-4
    staticip: 192.168.3.19
    fqdn: vcf-mgmt-esxi-4.home.lab
physical_esxi: # All the settings of your physical host.
  esxi_host: esxi1.home.lab
  esxi_username: root
  esxi_password: "PASSWORDSETME"
  datastore_name: Datastore1
nested_info_global: # Global settings for the nested ESXi VMs
  username: root
  password: "PASSWORDSETME"
  dns: 192.168.3.6
  domain: home.lab
  gateway: 192.168.3.1
  staticmask: 255.255.255.0
  ntpserver: pool.ntp.org
  networks:
    - name: "VM Network" # 1 of 2 required networks for management and VM networks
    - name: "VM Network" # 2 of 2 required networks for management and VM networks
    - name: "vSanvMotionPG" # 1 of 2 required networks for vSAN and vMotion networks
    - name: "vSanvMotionPG" # 2 of 2 required networks for vSAN and vMotion networks
  iso_location: "/home/matt/files/VMware-VMvisor-Installer-8.0U2-22380479.x86_64.iso" # Location on your jumpbox where you are hosting the vSphere ISO file
  esxi_cpu: 8
  esxi_mem_mb: 64000
```


/hosts - Not a required file. No need to update.

/roles/deployesxi/tasks/main.yaml - There's a few variables to set in the creds file for this script to work

```
datacenter: ha-datacenter - Modify this to your datacenter name
path: "ISOs/{{ nested_info.name }}.iso" - This requires a folder called "ISOs" to exist in your root directory of the datacenter. Modify it as needed.
```

```
```

## Usage
It is recommended that you always use an encrypted vault. The creds.yaml file needs to be encrypted.

To run the playbook:
```
export ANSIBLE_HOST_KEY_CHECKING=False; ansible-playbook main.yaml -i hosts
```
