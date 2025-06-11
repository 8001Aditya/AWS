# AWS Multi-VPC Networking with VPC Peering and S3 Gateway Endpoint

This project demonstrates how to set up two custom Virtual Private Clouds (VPCs) in AWS with private/public subnets, secure EC2 instances, VPC peering, and S3 access using a Gateway Endpoint — all deployed in the Oregon region (`us-west-2`).

---

## 📌 Table of Contents
- [Architecture](#architecture)
- [Objectives](#objectives)
- [Pre-requisites](#pre-requisites)
- [Step-by-Step Implementation](#step-by-step-implementation)
  - [1. Create Two Custom VPCs](#1-create-two-custom-vpcs)
  - [2. Create Public and Private Subnets](#2-create-public-and-private-subnets)
  - [3. Create and Attach Route Tables](#3-create-and-attach-route-tables)
  - [4. Launch EC2 Instances in Private Subnets](#4-launch-ec2-instances-in-private-subnets)
  - [5. Configure Security Groups](#5-configure-security-groups)
  - [6. Set Up VPC Peering](#6-set-up-vpc-peering)
  - [7. Create an S3 Gateway Endpoint](#7-create-an-s3-gateway-endpoint)
- [Testing](#testing)
- [Common Doubts & Clarifications](#common-doubts--clarifications)
- [Conclusion](#conclusion)

---

## 🏗️ Architecture

- **VPC-A**: `PRT-App-VPC (10.0.0.0/16)`
  - `Public Subnet-A`: `10.0.1.0/24`
  - `Private Subnet-A`: `10.0.2.0/24`
  - EC2: `App-Instance` in Private Subnet
- **VPC-B**: `PRT-Data-VPC (10.1.0.0/16)`
  - `Public Subnet-B`: `10.1.1.0/24`
  - `Private Subnet-B`: `10.1.2.0/24`
  - EC2: `Data-Instance` in Private Subnet
- **VPC Peering**: Enables communication between the two VPCs
- **S3 Gateway Endpoint**: Allows Data-Instance to access S3 privately

---

## 🎯 Objectives

- Create two custom VPCs with isolated networks
- Launch private EC2 instances inside each VPC
- Allow only inter-instance communication via Security Groups
- Enable VPC Peering between VPC-A and VPC-B
- Configure S3 Gateway Endpoint in VPC-B

---

## ⚙️ Pre-requisites

- AWS account with access to VPC, EC2, and S3
- IAM role/permissions for networking and EC2 actions
- Basic understanding of AWS networking (VPC, subnets, routing)

---

## 🔄 Step-by-Step Implementation

### 1. Create Two Custom VPCs

- Navigate to **VPC Dashboard > Your VPCs > Create VPC**
- Create `PRT-App-VPC` with CIDR: `10.0.0.0/16`
- Create `PRT-Data-VPC` with CIDR: `10.1.0.0/16`

---

### 2. Create Public and Private Subnets

**In PRT-App-VPC**:
- `Public Subnet-A`: `10.0.1.0/24`, AZ: us-west-2a
- `Private Subnet-A`: `10.0.2.0/24`, AZ: us-west-2a

**In PRT-Data-VPC**:
- `Public Subnet-B`: `10.1.1.0/24`, AZ: us-west-2b
- `Private Subnet-B`: `10.1.2.0/24`, AZ: us-west-2b

---

### 3. Create and Attach Route Tables

> 💬 **Doubt**: "In 3.2, why are we editing route tables without creating any?"
> ✅ **Clarification**: AWS creates one "main" RT per VPC. But for better control, we create separate ones manually.

**For PRT-App-VPC**:
- `App-Public-RT` → route `0.0.0.0/0 → App-IGW` → Associate with `Public Subnet-A`
- `App-Private-RT` → Associate with `Private Subnet-A`

**For PRT-Data-VPC**:
- `Data-Public-RT` → route `0.0.0.0/0 → Data-IGW` → Associate with `Public Subnet-B`
- `Data-Private-RT` → Associate with `Private Subnet-B`

---

### 4. Launch EC2 Instances in Private Subnets

- EC2 > Launch Instance
- AMI: Amazon Linux 2023
- Type: t2.micro
- **App-Instance**: In `Private Subnet-A`, SG: `App-SG`
- **Data-Instance**: In `Private Subnet-B`, SG: `Data-SG`

---

### 5. Configure Security Groups

**App-SG**:
- Outbound: Allow all to `10.1.0.0/16` (VPC-B)

**Data-SG**:
- Inbound: Allow only from App-Instance’s IP or App-SG ID

---

### 6. Set Up VPC Peering

- VPC > Peering Connections > Create Peering
  - Requester: `PRT-App-VPC`
  - Accepter: `PRT-Data-VPC`
- Accept peering connection
- Modify **Private Route Tables**:
  - App-Private-RT: `10.1.0.0/16 → Peering`
  - Data-Private-RT: `10.0.0.0/16 → Peering`

---

### 7. Create an S3 Gateway Endpoint

> 💬 **Doubt**: "Why am I seeing subnet selection instead of route tables?"
> ✅ **Clarification**: You likely selected an **Interface Endpoint**, not a **Gateway Endpoint**. S3 requires a Gateway Endpoint.

- Go to **VPC > Endpoints > Create Endpoint**
- Service: `com.amazonaws.us-west-2.s3` (Gateway type)
- VPC: `PRT-Data-VPC`
- Route Table: Select `Data-Private-RT`
- Policy: Full Access or custom

---

## ✅ Testing

- SSH into App-Instance using Bastion (optional)
- From App-Instance: ping or curl `Data-Instance` private IP
- On Data-Instance, run:

```bash
aws s3 ls
