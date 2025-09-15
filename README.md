# Active-Directory

## Objective

This lab aims to design and automate a Security Operations Center (SOC) lab that integrates Active Directory as the core identity infrastructure, simulating realistic enterprise environments. The project focuses on detecting, analyzing, and responding to adversary attack chains by automating log collection, correlation, and response workflows. This includes mapping simulated threats to MITRE ATT&CK techniques within Active Directory, automating detection pipelines, and orchestrating incident response to improve speed, accuracy, and resilience of security operations.

## Skills Learned

## Tools Used

## Procedure
1. Login to the Virtual Machine via RDP/SSH
2. Ubuntu server installation on Vultr
3. Wazuh installation
4. TheHive installation
5. Tools Configuration
6. Malware Analysis (Mimikatz)
7. Automation and enrichment

## Logical Diagram of the Setup

> [!NOTE]
> At the time of drafting this readme, Windows standard 2022 OS is not available in India location, so Atlanta, USA server is used.
> Server Setup is not included in this repo but the configuartions are. Refer to [SERVER SETUP](https://github.com/satyu-a/SOC-automation?tab=readme-ov-file#step-2-ubuntu-server-installation-on-vultr) for guide. Everything except the OS       selection is same. OS is **Windows standard 2022x64**.

## Machine Configurations:
1. Cloud Hosting: VULTR
2. Domain controller:
    - Hostname: DEFYER-ADDC-01
    - Specs: Atlnta USA location, Shared CPU, 2vCPU, 4GB RAM, 80GB SSD, Windows Standard 2022x64
    - VULTR firewall rule 1: Protocol - SSH, Port - 22, Action - Accept, Source - Custom, [Your_IP_Address] 
    - VULTR firewall rule 2: Protocol - MS RDP, Port - 3389, Action - Accept, Source - Custom, [Your_IP_Address]
2. Test Machine:
    - Hostname: DEFYER-ADTM-01
    - Specs: Atlnta USA location, Shared CPU, 1vCPU, 2GB RAM, 55GB SSD, Windows Standard 2022x64
    - VULTR firewall rule 1: Protocol - SSH, Port - 22, Action - Accept, Source - Custom, [Your_IP_Address] 
    - VULTR firewall rule 2: Protocol - MS RDP, Port - 3389, Action - Accept, Source - Custom, [Your_IP_Address]
3. Splunk Enterprise:
    - Hostname: DEFYER-ADSP-01
    - Specs: Atlnta USA location, Shared CPU, 4vCPU, 8GB RAM, 160GB SSD, Ubuntu 22.04
    - VULTR firewall rule 1: Protocol - SSH, Port - 22, Action - Accept, Source - Custom, [Your_IP_Address] 
    - VULTR firewall rule 2: Protocol - MS RDP, Port - 3389, Action - Accept, Source - Custom, [Your_IP_Address] 
