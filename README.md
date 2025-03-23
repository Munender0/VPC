# AWS VPC SETUP

This readme provides step-by-step instructions to set up an AWS VPC with the following architecture:

- A single **VPC**.
- Two **Availability Zones (AZs)**: **Zone A** and **Zone B**.
- Each AZ contains one **public subnet** and one **private subnet**:
  - **Zone A**: Public Subnet 1 and Private Subnet 1.
  - **Zone B**: Public Subnet 2 and Private Subnet 2.
- Public subnets are connected to the internet using an **Internet Gateway (IGW)**.
- Private subnets communicate securely via a **Virtual Private Gateway (VPG)**.

---

## Key Terminology

### Virtual Private Cloud (VPC)
A Virtual Private Cloud is a logically isolated portion of the AWS Cloud where resources can operate in a defined virtual network. It provides control over IP ranges, subnets, route tables, and gateways.

### Subnets
Subnets divide a VPC into smaller segments to group resources by IP range within an AZ. Public subnets enable external access, while private subnets are restricted from internet access.

### Internet Gateway (IGW)
An IGW enables resources in public subnets to communicate with the internet. It serves as the target for internet-bound traffic in public route tables.

### Virtual Private Gateway (VPG)
A VPG acts as a secure VPN concentrator, facilitating communication between private subnets and external private networks such as on-premises data centers or other VPCs.

### Route Table
A route table is a set of rules, or "routes," that determines the flow of network traffic within a VPC.

### Availability Zone (AZ)
An AZ is a distinct geographical location within an AWS Region, designed for fault isolation and high availability.

---

## Architecture Summary

### VPC Configuration
- **CIDR Block**: `10.0.0.0/24`

### Subnet Configuration

| Zone        | Public Subnet       | Private Subnet       |
|-------------|---------------------|----------------------|
| **Zone A**  | `10.0.0.0/26`       | `10.0.0.128/26`      |
| **Zone B**  | `10.0.0.64/26`      | `10.0.0.192/26`      |

### Gateways
- **Internet Gateway (IGW)**: Connects public subnets to the internet.
- **Virtual Private Gateway (VPG)**: Enables private subnets to communicate securely with each other.

---

## Step-by-Step Implementation

### Step 1: VPC Creation
1. Log in to the **AWS Management Console**.
2. Go to the **VPC Dashboard** and click **Create VPC**.
3. Configure the VPC:
   - **Name**: `My-VPC`
   - **CIDR Block**: `10.0.0.0/24`
   - Leave other settings as default.
4. Click **Create VPC**.

![Screenshot (23)](https://github.com/user-attachments/assets/0861ff06-a004-40a1-b974-2b50637dd40b)



### Step 2: Subnet Creation

#### Zone A
1. Navigate to **Subnets** in the VPC Dashboard.
2. Create `Public-Subnet-1`:
   - **VPC**: Select `My-VPC`.
   - **AZ**: Choose `us-east-1a`.
   - **CIDR Block**: `10.0.0.0/26`.
3. Create `Private-Subnet-1`:
   - **VPC**: `My-VPC`.
   - **AZ**: `us-east-1a`.
   - **CIDR Block**: `10.0.0.128/26`.

#### Zone B
1. Create `Public-Subnet-2`:
   - **VPC**: `My-VPC`.
   - **AZ**: `us-east-1b`.
   - **CIDR Block**: `10.0.0.64/26`.
2. Create `Private-Subnet-2`:
   - **VPC**: `My-VPC`.
   - **AZ**: `us-east-1b`.
   - **CIDR Block**: `10.0.0.192/26`.

![Screenshot (24)](https://github.com/user-attachments/assets/a191d8ab-a54b-4c66-ba2c-babcaa085a14)



### Step 3: Internet Gateway (IGW)
1. In the **VPC Dashboard**, go to **Internet Gateways** and click **Create Internet Gateway**.
2. Configure the IGW:
   - **Name**: `My-IGW`
3. Attach the IGW to `My-VPC`:
   - Select the IGW, then click **Actions** → **Attach to VPC** → Choose `My-VPC`.

![Screenshot (25)](https://github.com/user-attachments/assets/77493e08-738b-4fd7-8a42-f2ab5c328196)



### Step 4: Virtual Private Gateway (VPG)
1. Navigate to **Virtual Private Gateways** and click **Create Virtual Private Gateway**.
2. Configure the VPG:
   - **Name**: `My-VPG`.
3. Attach the VPG to `My-VPC`:
   - Select the VPG → **Actions** → **Attach to VPC** → Choose `My-VPC`.

![Screenshot (26)](https://github.com/user-attachments/assets/d78645d6-1571-4db7-8c84-6993d032a53c)



### Step 5: Configure Route Tables

#### Public Route Table
1. Create a public route table:
   - Go to **Route Tables** and click **Create Route Table**.
   - **Name**: `Public-Route-Table`.
   - **VPC**: `My-VPC`.
2. Add a route for internet access:
   - Go to the **Routes** tab → **Edit Routes** → Add:
     - **Destination**: `0.0.0.0/0`.
     - **Target**: `My-IGW`.
3. Associate public subnets:
   - In the **Subnet Associations** tab → **Edit Subnet Associations** → Select `Public-Subnet-1` and `Public-Subnet-2`.


![Screenshot (27)](https://github.com/user-attachments/assets/6e3ae06f-31ca-440c-b5b3-61f213148ce8)

#### Private Route Table
1. Create a private route table:
   - **Name**: `Private-Route-Table`.
   - **VPC**: `My-VPC`.
2. Enable VPG route propagation:
   - In the **Route Propagation** tab → **Edit Route Propagation** → Select `My-VPG`.
3. Associate private subnets:
   - In **Subnet Associations** → **Edit Subnet Associations** → Select `Private-Subnet-1` and `Private-Subnet-2`.

![Screenshot (28)](https://github.com/user-attachments/assets/4a979b91-7e6b-4572-bce0-b50352ca8866)



### Step 6: Launch EC2 Instances

#### Public Subnets
1. Launch an EC2 instance in `Public-Subnet-1`:
   - Assign a **Public IP Address**.
   - Use a security group allowing SSH (port 22).
2. Repeat for `Public-Subnet-2`.

![Screenshot (29)](https://github.com/user-attachments/assets/79bc913a-c1cf-4dbd-93da-d8a208388771)



#### Private Subnets
1. Launch an EC2 instance in `Private-Subnet-1`:
   - Disable **Public IP Address**.
2. Repeat for `Private-Subnet-2`.

---

## Configure SSH Access for Instances
1. Navigate to **Security Groups** → **Create Security Group**:
   - **Name**: `Private-SSH-Access`.
   - Add **Inbound Rule**:
     - **Type**: `SSH`.
     - **Port**: `22`.
     - **Source**: Use your IP (`My IP`) or a specific CIDR.

![Screenshot (30)](https://github.com/user-attachments/assets/f257ac80-9427-42d9-8870-214d8b2a00d8)


       
2. Attach this security group to private instances.

---

## Validation
- Confirm public instances can access the internet.

![Screenshot (33)](https://github.com/user-attachments/assets/29ac5943-e953-4251-869a-b0c6fabe2c3d)


  
- Verify private instances are isolated and connected through the VPG.

---

## References
- [AWS VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/latest/userguide/)
- [AWS Internet Gateway Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
- [AWS Virtual Private Gateway Documentation](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html)
