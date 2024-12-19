# Documentation: Setting Up a Bastion Host in AWS
##### This document outlines the steps to set up a Bastion Host environment in AWS. The purpose of this setup is to provide secure access to resources in a private subnet using a Bastion Host deployed in a public subnet.

# Step 1: Create a VPC
**Action**: Create a new Virtual Private Cloud (VPC).
**Details**: Configure the VPC CIDR block based on your network design (e.g., 10.0.0.0/16).
# Step 2: Create Subnets
#### Public Subnet:
**CIDR Block**: Select a range within the VPC CIDR (e.g., 10.0.1.0/24).
**Purpose**: Hosts the Bastion Host, enabling public access.
#### Private Subnet:
**CIDR Block**: Select a separate range within the VPC CIDR (e.g., 10.0.2.0/24).
**Purpose**: Hosts the production server, isolated from public access.
# Step 3: Create an Internet Gateway
**Action**: Create and attach an Internet Gateway (IGW) to the VPC.
**Purpose**: Enables internet access for resources in the public subnet.
# Step 4: Create a NAT Gateway
**Action**: Create NAT Gateway and select the Private subnet.
**Elastic IP**: Allocate an Elastic IP address for the NAT Gateway.
**Purpose**: Allows resources in the private subnet to access the internet securely for updates and patches.
# Step 5: Configure Route Tables
#### Public Route Table:
**Routes**: Click on edit routes and Add a route with destination 0.0.0.0/0 pointing to the Internet Gateway.
**Subnet Association**: Associate the public subnet with this route table.
#### Private Route Table:
**Routes**: Click on edit route and Add a route with destination 0.0.0.0/0 pointing to the NAT Gateway.
**Subnet Association**: Associate the private subnet with this route table.
# Step 6: Create Security Groups
#### Public Security Group:
**Inbound Rule**: Allow access from our IP address (e.g., MY IP).
**Purpose**: Secures the Bastion Host, restricting access to trusted IPs only.
#### Private Security Group:
**Inbound Rule**: Allow access only from the public security group.
**Purpose**: Secures the production server, permitting access exclusively from the Bastion Host.
# Step 7: Launch EC2 Instances
#### Bastion Host:
**Subnet**: Public Subnet.
**Auto-Assign Public IPv4**: Enabled.
**Security Group**: Attach the public security group.
**Purpose**: Acts as a gateway for secure SSH access to private resources.
#### Production Server:
**Subnet**: Private Subnet.
**Auto-Assign Public IPv4**: Disabled.
**Security Group**: Attach the private security group.
**Purpose**: Hosts your production applications or services, isolated from direct internet access.
# Step 8: Connecting to the Bastion Host and Production Server
#### 1. Connect to the Bastion Host
To access the Bastion Host, use the following SSH command:
```bash
ssh -i "/path/to/public.pem" ec2-user@<public_ip_of_bastion_host>
```
Replace **<public_ip_of_bastion_host>** with the public IP address of your Bastion Host and /path/to/public.pem with the path to your public key file.
#### 2. Forward the Private Key for Production Server Access
Since the Bastion Host does not have the private key required to access the Production Server, use SSH Agent Forwarding:
#### Add the private key to your local SSH agent:
```bash
ssh-add /path/to/private.pem
```

Replace **/path/to/private.pem** with the path to your private key file.
#### Connect to the Bastion Host with Agent Forwarding enabled:
```bash
ssh -A -i "/path/to/public.pem" ec2-user@<public_ip_of_bastion_host>
```
#### 3. Access the Production Server from the Bastion Host
Once logged into the Bastion Host, you can SSH into the Production Server using its private IP address:
```bash
ssh ec2-user@<private_ip_of_production_server>
```
Replace **<private_ip_of_production_server>** with the private IP address of your Production Server.
# Notes:
- Ensure that the private key (private.pem) is added to your local machine's SSH agent before connecting to the Bastion Host.
- Ensure that the Bastion Host security group allows SSH access only from your IP address.
- The Production Server’s security group must allow SSH access from the Bastion Host's private IP address.
