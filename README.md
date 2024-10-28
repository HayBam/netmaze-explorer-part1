# Azure Hybrid Network Environment Setup

This repository provides step-by-step instructions to set up a simulated hybrid network environment in Microsoft Azure. This setup includes creating two Azure Virtual Networks (VNets) to represent a main network(MainVNet) and an on-premises network(HQVNet), configuring a VPN gateway for site-to-site connectivity, implementing secure access and network segmentation, deploying resources, and testing connectivity and security. This project will be split into two parts, this is part one of the project.

## Overview

This guide creates a hybrid network simulation in Azure using a secure site-to-site VPN to connect a primary Azure Virtual Network (VNet) with a secondary Azure VNet, representing an on-premises network.

### Overall Objectives
**Part 1**
- Create a multi-subnet named MainVNet with isolated environments for application, database, and admin resources.
- Simulate an on-premises network using a second VNet named HQVNet
- Establish secure connectivity with VPN Gateway using Site-to-Site connectivity and Network Security Groups (NSGs).
**Part 2**
- Implement Azure Bastion for secure RDP/SSH access to VMs without exposing public IPs.
- Implement Azure Load Balancer to distribute traffic to WebApp VMs.
- Implement private connectivity to Azure PaaS services using Private Link.
- Monitor and log network operations to maintain security and visibility.

## Architecture

The architecture consists of:
1. **Main Azure VNet** with subnets for WebApp, Database, and Admin resources.
2. **Simulated On-Premises VNet** with a single subnet, connected to the main VNet via site-to-site VPN.
3. **VPN Gateways** on both VNets to provide secure connectivity.
4. **NSGs** for secure traffic control within and between subnets.

## Prerequisites

1. **Azure Account** with the necessary permissions.
2. **Understanding of networking** with the understanding of subnetting 

---

## Detailed Setup Guide

### 1. Main Azure Virtual Network (MainVNet) Setup

The main Azure VNet(MainVNet) will host subnets for web apps, db, and admin access. Using separate subnets allows for network isolation, ensuring that traffic control rules can be customized per subnet.

#### Steps

1. **Create the Main VNet**:
   - Go to the Azure Portal (https://portal.azure.com).
   - Search for "Virtual Networks" and click "Create" to initiate a new VNet setup.
   - **Subscription**: Choose the active subscription.
   - **Resource Group**: Create or select a group.
   - **Name**: Enter `MainVNet`.
   - **Region**: Select the region in my case I used Canada Central.

2. **Specify IP Address Space**:
   - Set the address space as `10.1.0.0/16` to define a private IP range that accommodates multiple subnets.

3. **Create Subnets**:
   - Add subnets under the "IP Addresses" tab to isolate resources within the network.
   - **WebApp Subnet**: Define as `10.1.10.0/24`, designated for web server VMs.
   - **DB Subnet**: Define as `10.1.20.0/24`, designated for database resources.
   - **Admin Subnet**: Define as `10.1.30.0/24`, designated for administrative access.

  ![image](https://github.com/user-attachments/assets/0a78370e-9cdf-4d0f-8abd-b5142c42fd51)
  ![image](https://github.com/user-attachments/assets/7b50453a-568a-426f-a25c-67703dbace5e)
  ![image](https://github.com/user-attachments/assets/8f170caf-c5f1-433b-bea1-e0ea5030ea60)


4. **Review and Deploy**:
   - Click "Review + Create," and once validated, click "Create" to deploy the Main VNet.

### 2. Simulated On-Premises Network Setup (HQVNet)

To simulate an on-premises network environment, create a second VNet with a different IP range to avoid IP conflicts. Later, this network will connect to the MainVNet through a site-to-site VPN.

#### Steps

1. **Create On-Premises VNet**:
   - Repeat the steps to create a VNet, but with a new name and IP range:
   - **Name**: Set as `HQVNet`.
   - **Region**: Use the same region as Main VNet or a different one if desired.
   - **IP Address Space**: Define as `192.168.10.0/16` to simulate a separate on-prem network.

2. **Add Subnet**:
   - Create a single subnet named `OnPremSubnet` with IP range `192.168.10.0/24`.
   - This subnet will represent resources in the on-premises network.

3. **Deploy VNet**:
   - Review and create the VNet.

  ![image](https://github.com/user-attachments/assets/51d4e073-770c-467b-be27-715167734ad7)
  ![image](https://github.com/user-attachments/assets/63ffabf9-2632-477a-8c33-1612f737d408)
  ![image](https://github.com/user-attachments/assets/aef133ea-89a4-4cf3-b269-4d481df0d84d)


### 3. Secure Connectivity with VPN Gateway
Creating a Gateway subnet to allow the creation of a VPN gateway. We could achieve the connection with a VNet peering since both subnets are in Azure, however, we are trying to simulate an On-Prem to Azure connection, and this is only achievable with a site-to-site connection. 
To securely connect `MainVNet` and `HQVNet`, VPN Gateways are deployed on each network, and a site-to-site connection is configured.
#### Steps
Gateway
VPN
Local
1. **Create Gateway Subnet in MainVNet**:
   - Go to `MainVNet` and click on "Subnets."
   - Add a **Gateway Subnet** using an IP range, I used `10.1.255.0/27`.
  ![image](https://github.com/user-attachments/assets/32dc261f-79f0-406a-a22f-1b0e84b8bf83)

2. **Provision VPN Gateway for MainVNet**:
   - Search for "Virtual Network Gateway" in the portal and click "Create."
   - **Name**: Main-to-HQ
   - **Gateway Type**: Choose **VPN**.
   - **Region**: Select the same region for the MainVNet.
   - **SKU**: This is dependent on your requirement. I used the cheapest as this is not for production. I used `VpnGw1AZ`.
   - **Generation**: Select **Generation 1**, this is based on the SKU I used above
   - **Public IP Address**: Created a new IP address and  Public IP address name MainVNet-S2S-IP
   - **Virtual Network**: Select `MainVNet`.
   - **Subnet**: This will be automatically selected.
   - **Availability Zone**: I select `1` here as I don't need any redundancies.
   - **Anable active- active mode**: Disabled.
   - **Configure BGP**: Disabled.
   - **Address Space(s)**: Add the subents in the VNets for MainVNET I added Select `MainVNet`.
   - Deploying the gateway may take up to 45 minutes.
  ![image](https://github.com/user-attachments/assets/9d1345ad-cdd1-4f03-944c-dedbd1b0aad1)


3. **Create Gateway Subnet and VPN Gateway in On-Premises VNet**:
   - Go to `HQVNet` and click on "Subnets."
   - Add a **Gateway Subnet** using an IP range, I used `192.168.11.0/24` and a VPN gateway for `HQVNet`.
  ![image](https://github.com/user-attachments/assets/c97b2184-ac44-4a9e-b040-46e3c13d3d8f)

4. **Provision VPN Gateway for MainVNet**:
   - Search for "Virtual Network Gateway" in the portal and click "Create."
   - **Name**: HQ-to-Main
   - **Gateway Type**: Choose **VPN**.
   - **Region**: Select the same region for the MainVNet.
   - **SKU**: This is dependent on your requirement. I used the cheapest as this is not for production. I used `VpnGw1AZ`.
   - **Generation**: Select **Generation 1**, this is based on the SKU I used above
   - **Public IP Address**: Created a new IP address and  Public IP address name HQVNet-S2S-IP
   - **Virtual Network**: Select `HQVNet`.
   - **Subnet**: This will be automatically selected.
   - **Availability Zone**: I select `1` here as I don't need any redundancies.
   - **Anable active- active mode**: Disabled.
   - **Configure BGP**: Disabled.
   - **Address Space(s)**: Add the subents in the VNets for MainVNET I added Select `MainVNet`.
   - Deploying the gateway may take up to 45 minutes.
  ![image](https://github.com/user-attachments/assets/b7ca7653-7d06-42e4-8c7e-d0944c50820e)
  ![image](https://github.com/user-attachments/assets/b7d1dac3-71ca-4618-9293-d7e8d063cc20)
     
5. **Create Local Network Gateway**:
   - Navigate to "Local Network Gateway" and click "Create". Two local network Gateways would be created for each VNets
   - **Name**: Use `MainVNet-IP and HQVNet-IP`.
   - **IP Address**: For MainVNet-IP **public IP** of the `MainVNet` VPN Gateway while for HQVNet-IP **public IP** of the `HQVNet`
   - **Address Space**: Add the subents for each VNets as shown below.
   - This gateway represents the on-premises network configuration.
  ![image](https://github.com/user-attachments/assets/d5c07678-4bcc-47d7-84e8-df94ca06e717)
  ![image](https://github.com/user-attachments/assets/d1d52a13-08f3-4cc9-8e35-9ca053633783)
  ![image](https://github.com/user-attachments/assets/d9bcfe49-d467-4b16-858a-5a335d84ff9c)

6. **Establish Site-to-Site VPN Connection**:
   - Go to `MainVNetGateway` > "Connections" > "Add."
   - **Connection Type**: Select **Site-to-Site (IPSec)**.
   - **Shared Key**: Enter a strong key for encryption.
   - **Local Network Gateway**: Select `OnPremLocalNetworkGateway`.
   - Repeat these steps for the `HQVNet` gateway to complete the two-way connection.
  ![image](https://github.com/user-attachments/assets/0f83a07a-15ac-449b-9d32-2792e4ac6e8b)
  ![image](https://github.com/user-attachments/assets/b3b5b996-8aea-4c50-96a9-2e415e58b5de)

7. **Verify VPN Connectivity**:
   - Deploy VMs in `MainVNet` and `HQVNet` to test connectivity (e.g., using `ping` or `tracert`).
   - Ensure NSG rules allow ICMP or RDP/SSH traffic for testing.
   - I deployed two(mainvm1 and hqvm1) linux servers in each VNet for the test
   ![image](https://github.com/user-attachments/assets/f41ae12e-3274-42de-8500-5e0c7eb2078a)

Able to ping main-vm1(Webapp VM) private IP from hqvm1
  ![image](https://github.com/user-attachments/assets/c2483190-b40c-49d1-a72a-f867d39e2855)

Can access the website deployed on main-vm1(Webapp VM)
  ![image](https://github.com/user-attachments/assets/83673140-4c66-4d88-97b3-23e8fcf0a175)


### 4. Deploy Resources in Subnets

Deploy test VMs in each subnet to simulate different workloads, such as web servers and databases, and to validate network configurations.

1. **Deploy VM in WebApp Subnet**:
   - Go to "Virtual Machines" and click "Create."
   - Select `MainVNet` and `WebApp Subnet` for the network configuration.
   - Complete other VM configuration details as per requirement.
  ![image](https://github.com/user-attachments/assets/9a074d1d-6234-4971-a487-6472460975ae)

2. **Deploy VM in Database Subnet**:
   - Repeat the process, selecting `Database Subnet`.

3. **Deploy VM in Admin Subnet**:
   - Similarly, deploy a VM in `Admin Subnet` to enable administrative access.

### 5. Network Access Control using NSGs

Use NSGs to control traffic flow between subnets.

1. **Create NSG for WebApp Subnet**:
   - Go to "Network Security Groups" and create `WebAppNSG`.
   - **Inbound Rules**: Allow HTTP (80) and HTTPS (443) traffic from the internet.
   - **Outbound Rules**: Allow traffic to `Database Subnet`.
WebApp NSG
![image](https://github.com/user-attachments/assets/6501c87c-2559-41e8-a368-c2e834e56d6c)
HQVM NSG
![image](https://github.com/user-attachments/assets/04c21f2c-eace-40a4-9061-703ba2b2d989)

2. **Create NSG for Database Subnet**:
   - Create `DBNSG` and allow inbound traffic only from `WebApp Subnet` IP range.

3. **Create NSG for Admin Subnet**:
   - Create `AdminNSG` to restrict access only to trusted IPs.

In Part 2 we shall work on 
**Azure Bastion** for secure RDP/SSH access to VMs without exposing public IPs.
**Azure Private Link** for private access to Azure PaaS resources.
**Azure Load Balancer** to distribute traffic to WebApp VMs.

---

## References
- The idea for this project was gotten from madebygps - https://github.com/madebygps/projects/blob/main/az-104/netmazeexplorer.md
- Virtual Network documentation - https://learn.microsoft.com/en-us/azure/virtual-network/
- Create and manage a VPN gateway using the Azure portal - https://learn.microsoft.com/en-us/azure/vpn-gateway/tutorial-create-gateway-portal
