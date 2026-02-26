# [WATCH ME BUILD THE LAB HERE](https://www.loom.com/share/7d38985af54f49c6a4d7890e9beb720c)

# Lab 02: Building-a-Secure-2-Tier-Web-Application
This lab provides a hands -on, step-by-step guide to designing and deploying a secure 2-tier web application architecture, emphasizing, access control, and infrastructure security best practices.

## Overview

This lab demonstrates how to design and deploy a secure 2-tier Infrastructure-as-a-Service (IaaS) architecture in Microsoft Azure.

The environment includes:

- A Public Web Server  
- A Private Database Server  
- A Virtual Network (VNet) with segmented subnets  
- Network Security Groups (NSGs) enforcing least-privilege access  

The goal is to ensure:

- The Web Server is accessible from the internet  
- The Database Server is not publicly accessible  
- Only the Web Server can communicate with the Database Server  

---

## Architecture Design

        Internet
            |
            v
    -----------------
    |  Public Subnet |
    |   vm-web-01    |
    -----------------
            |
    Internal VNet Traffic
            |
            v
    -----------------
    | Private Subnet |
    |   vm-db-01     |
    -----------------

---

## Key Security Principles Applied

- Network Segmentation  
- Least Privilege Access  
- Private Backend Infrastructure  
- Controlled Internal Communication  

---

## Prerequisites

- Active Azure Subscription  
- Access to Azure Portal or Azure CLI  
- SSH key pair  
- Basic understanding of Virtual Networks and Subnets  

---

## Resource Naming Convention

- **Resource Group:** rg-lab02-glenpage  
- **Virtual Network:** vnet-lab02  
- **Address Space:** 10.0.0.0/16  
- **Public Subnet:** snet-web (10.0.1.0/24)  
- **Private Subnet:** snet-db (10.0.2.0/24)  
- **Web VM:** vm-web-01  
- **Database VM:** vm-db-01  

---

## Phase 1: Create the Network Foundation

- Create a Resource Group in East US  
- Create a Virtual Network with address space 10.0.0.0/16  
- Create two subnets:
  - snet-web → 10.0.1.0/24  
  - snet-db → 10.0.2.0/24  

This establishes segmentation between public-facing and private workloads.

---

## Phase 2: Deploy the Web Server (Public Tier)

Create a Virtual Machine with the following configuration:

- **Name:** vm-web-01  
- **Image:** Ubuntu Server 20.04 LTS  
- **Size:** Standard_B1s  
- **Subnet:** snet-web  
- **Public IP:** Enabled  
- **Inbound Ports Allowed:**
  - SSH (22)
  - HTTP (80)

After deployment:

- Retrieve the Public IP  
- Verify SSH connectivity  

Example SSH command:


ssh -i key-lab02.pem azureuser@<public-ip>


---

## Phase 3: Deploy the Database Server (Private Tier)

Create a second Virtual Machine:

- **Name:** vm-db-01  
- **Image:** Ubuntu Server 20.04 LTS  
- **Size:** Standard_B1s  
- **Subnet:** snet-db  
- **Public IP:** Disabled  

After deployment:

- Retrieve the Private IP address  
- Confirm it belongs to 10.0.2.0/24  

---

## Phase 4: Validate Internal Connectivity (Jump Method)

Because the database server has no Public IP:

1. SSH into the Web Server  
2. From inside vm-web-01, ping the private IP of vm-db-01  

Example:


ping 10.0.2.4


If successful, this confirms:

- Both VMs are in the same VNet  
- Internal routing works  
- Subnet segmentation allows internal communication  

---

## Phase 5: Configure Database Security (NSG Rules)

Create an inbound NSG rule:

- **Source:** 10.0.1.0/24 (Web Subnet)  
- **Destination:** Any  
- **Protocol:** TCP  
- **Port:** * (or 3306 / 5432 if using a database)  
- **Action:** Allow  
- **Priority:** 100  

Security outcome:

- Only the Web Subnet can access the Database Server  
- Internet traffic remains blocked  
- Backend infrastructure stays isolated  

---

## Security Validation Checklist

- Web VM has a Public IP  
- DB VM does NOT have a Public IP  
- DB VM resides in snet-db  
- NSG rule allows only Web Subnet traffic  
- Internal ping from Web → DB succeeds  

---

## Troubleshooting

### Cannot SSH into Database VM

This is expected. The DB VM does not have a Public IP.

Use the Web VM as a jump host.

### Ping Fails Between Servers

Verify:

- Both VMs are in the same VNet  
- DB VM is deployed to snet-db  
- NSG rules allow internal traffic  
- No conflicting deny rules exist  

---

## Clean Up

To avoid charges:

- Navigate to Resource Groups  
- Select rg-lab02-glenpage  
- Click Delete Resource Group  

This removes all associated infrastructure.
