Lab 02: Building a Secure 2-Tier Web Application

Author: Glen Page
Estimated Time: 60 Minutes
Difficulty: Beginner

Overview

This lab demonstrates how to build a classic Infrastructure as a Service (IaaS) 2-tier architecture in Azure. You will deploy:

A Public Web Tier hosting a web server

A Private Data Tier hosting a database server

Network Security Groups (NSGs) are used to ensure the database server is isolated from the internet and can only be accessed by the web server.

Architecture Diagram




Traffic Flow:

Internet → Web Server (Public Subnet)

Web Server → Database Server (Private Subnet)

The database server has no public IP and is reachable only from within the virtual network.

Prerequisites

Active Azure subscription

Access to the Azure Portal

Terminal (macOS, Linux, or Windows with WSL)

Basic understanding of virtual networks and virtual machines

Lab Variables (Naming Convention)
Resource	Value
Resource Group	rg-lab02-[yourname]
Virtual Network	vnet-lab02
Public Subnet	snet-web (10.0.1.0/24)
Private Subnet	snet-db (10.0.2.0/24)
Web VM	vm-web-01
Database VM	vm-db-01
Step-by-Step Instructions
Phase 1: The Network Foundation




Search for Virtual Networks → Create

Under Basics:

Resource Group: Create new → rg-lab02-[yourname]

Name: vnet-lab02

Region: East US

Under IP Addresses:

Address space: 10.0.0.0/16

Subnet 1:

Name: snet-web

Range: 10.0.1.0/24

Subnet 2:

Name: snet-db

Range: 10.0.2.0/24

Click Review + create → Create

Phase 2: Deploying the Web Server (Front End)




Search for Virtual Machines → Create

Under Basics:

Resource Group: rg-lab02-[yourname]

Name: vm-web-01

Region: East US

Image: Ubuntu Server 20.04 LTS

Size: Standard_B1s

Authentication: SSH public key

Key pair name: key-lab02

Public inbound ports: Allow selected → HTTP (80), SSH (22)

Under Networking:

Virtual Network: vnet-lab02

Subnet: snet-web

Public IP: Create new (Standard)

Click Review + create → Create

Download the private key (.pem) when prompted

Phase 3: Deploying the Database Server (Back End)




Create another Virtual Machine

Under Basics:

Resource Group: rg-lab02-[yourname]

Name: vm-db-01

Region: East US

Image: Ubuntu Server 20.04 LTS

Size: Standard_B1s

Key pair: Use existing key → key-lab02

Public inbound ports: Allow selected → SSH (22)

Under Networking (Critical Step):

Virtual Network: vnet-lab02

Subnet: snet-db

Public IP: None

Click Review + create → Create

This virtual machine is fully private and not accessible from the internet.

Phase 4: Validating Connectivity (Jump Host Access)




Because vm-db-01 has no public IP, you must connect through the Web Server.

Get the Private IP of the Database Server

Open vm-db-01 in the Azure Portal

Copy the Private IP address (example: 10.0.2.4)

SSH into the Web Server
ssh -i key-lab02.pem azureuser@[public-ip-of-web]
Test Internal Connectivity
ping 10.0.2.4

Successful replies confirm private subnet connectivity. Press Ctrl + C to stop the ping.

Phase 5: Configuring the Firewall (Network Security Group)




Open vm-db-01 → Networking

Click the attached Network Security Group

Select Inbound security rules → + Add

Create the "Allow Web Subnet" Rule

Source: IP Addresses

Source IP ranges: 10.0.1.0/24

Source ports: *

Destination: Any

Service: Custom

Destination ports: * (or 3306 / 5432 for real databases)

Action: Allow

Priority: 100

Name: Allow-Web-Subnet

Click Add.

The absence of a Public IP already blocks internet traffic. This rule explicitly allows internal traffic from the web tier.

Troubleshooting

Issue: Ping fails
Fix: Ensure both VMs are in the same virtual network and vm-db-01 is deployed in snet-db.

Issue: Cannot SSH into vm-db-01 from home computer
Reason: The VM has no public IP. You must SSH into vm-web-01 first.

Clean Up

To avoid ongoing charges:

Delete the resource group: rg-lab02-[yourname]

This will remove all resources created during the lab.