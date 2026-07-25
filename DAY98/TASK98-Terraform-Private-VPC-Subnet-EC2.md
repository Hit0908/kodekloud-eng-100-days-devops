# 🌟 Task 98 - Create Private VPC + Subnet + EC2 with Internal-Only Access

**📌 Task Description**  
The **Nautilus DevOps team** is building a **secure, private AWS environment**:

**Goal**:
- **VPC**: `devops-priv-vpc` → `10.0.0.0/16`  
- **Subnet**: `devops-priv-subnet` → `10.0.1.0/24` → **No public IP**  
- **EC2**: `devops-priv-ec2` → `t2.micro` → **Private only**  
- **Security Group**: Allow only VPC CIDR (`10.0.0.0/16`)  
- Use `variables.tf`, `outputs.tf`  
- All in `/home/bob/terraform`

---

## 📋 Requirements Summary

| **Resource** | **Name** | **CIDR** | **Public IP** | **SG Rule** |
|--------------|----------|----------|---------------|-------------|
| VPC | devops-priv-vpc | 10.0.0.0/16 | — | — |
| Subnet | devops-priv-subnet | 10.0.1.0/24 | Disabled | — |
| EC2 | devops-priv-ec2 | — | — | VPC CIDR only |

---

## 📝 File Structure
```
/home/bob/terraform/
├── main.tf
├── variables.tf
├── terraform.tfvars
└── outputs.tf
```

---

## 🔹 Implementation Steps

### **Step 1: Connect to Jump Host**
```bash
ssh bob@jumphost
```

---

### **Step 2: Navigate to Terraform Directory**
```bash
cd /home/bob/terraform
```

---

### **Step 3: Create `main.tf`**
```bash
vi main.tf
```

**Content**:  
```hcl
# Configure AWS Provider
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1" # Adjust region if required by your environment
}

# 1. VPC
resource "aws_vpc" "devops_priv_vpc" {
  cidr_block           = var.KKE_VPC_CIDR
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "devops-priv-vpc"
  }
}

# 2. Subnet
resource "aws_subnet" "devops_priv_subnet" {
  vpc_id                  = aws_vpc.devops_priv_vpc.id
  cidr_block              = var.KKE_SUBNET_CIDR
  map_public_ip_on_launch = false

  tags = {
    Name = "devops-priv-subnet"
  }
}

# 3. Security Group (Allows inbound traffic only from within the VPC CIDR)
resource "aws_security_group" "devops_priv_sg" {
  name        = "devops-priv-sg"
  description = "Allow inbound traffic from VPC CIDR only"
  vpc_id      = aws_vpc.devops_priv_vpc.id

  ingress {
    description = "Allow traffic from within VPC CIDR"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = [var.KKE_VPC_CIDR]
  }

  egress {
    description = "Allow all outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "devops-priv-sg"
  }
}

# Fetch latest Amazon Linux 2 AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# 4. EC2 Instance
resource "aws_instance" "devops_priv_ec2" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.devops_priv_subnet.id
  vpc_security_group_ids = [aws_security_group.devops_priv_sg.id]

  tags = {
    Name = "devops-priv-ec2"
  }
}
```

---

### **Step 4: Create `variables.tf`**
```bash
vi variables.tf
```

**Content**:  
```hcl
variable "KKE_VPC_CIDR" {
  type        = string
  default     = "10.0.0.0/16"
  description = "CIDR block for devops-priv-vpc"
}

variable "KKE_SUBNET_CIDR" {
  type        = string
  default     = "10.0.1.0/24"
  description = "CIDR block for devops-priv-subnet"
}
```

---
---

### **Step 6: Create `outputs.tf`**
```bash
vi outputs.tf
```

**Content**:  
```hcl
output "KKE_vpc_name" {
  value       = aws_vpc.devops_priv_vpc.tags["Name"]
  description = "Name of the VPC"
}

output "KKE_subnet_name" {
  value       = aws_subnet.devops_priv_subnet.tags["Name"]
  description = "Name of the Subnet"
}

output "KKE_ec2_private" {
  value       = aws_instance.devops_priv_ec2.tags["Name"]
  description = "Name of the EC2 Instance"
}
```

---

### **Step 7: Run Terraform**
```bash
terraform init
terraform validate
terraform plan
terraform apply -auto-approve
```

**Expected Output**:  
```
+ create aws_vpc.devops_priv_vpc
+ create aws_subnet.devops_priv_subnet
+ create aws_security_group.devops_priv_sg
+ create aws_instance.devops_priv_ec2

Apply complete! Resources: 4 added.
```

---

## 🔍 Verification

### **Check EC2 Instance**
```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=xfusion-priv-ec2" \
  --query "Reservations[].Instances[].{PublicIp:PublicIpAddress,PrivateIp:PrivateIpAddress}"
```

**Expected**:  
```json
[
  {
    "PublicIp": null,
    "PrivateIp": "10.0.1.x"
  }
]
```

### **Check Security Group Rules**
```bash
aws ec2 describe-security-groups \
  --filters Name=tag:Name,Values=xfusion-priv-sg \
  --query "SecurityGroups[].IpPermissions[].{From:FromPort,To:ToPort,CIDR:IpRanges[].CidrIp}"
```

**Expected**:  
```json
[
  {
    "From": 0,
    "To": 0,
    "CIDR": ["10.0.0.0/16"]
  }
]
```

---

## 📊 Code Analysis

| **Feature** | **Implementation** |
|-------------|-------------------|
| Private Subnet | `map_public_ip_on_launch = false` |
| Internal SG | `cidr_blocks = [var.KKE_VPC_CIDR]` |
| Variables | `KKE_VPC_CIDR`, `KKE_SUBNET_CIDR` |
| Outputs | `KKE_vpc_name`, `KKE_subnet_name`, `KKE_ec2_private` |

---

## 📖 Quick Command Reference
```bash
cd /home/bob/terraform

# Run
terraform init
terraform plan
terraform apply -auto-approve

# Verify
terraform output
```

---

## 💡 Common Issues & Fixes

| **Issue** | **Fix** |
|-----------|---------|
| Public IP assigned | `map_public_ip_on_launch = false` |
| SG allows internet | Use `var.KKE_VPC_CIDR` |
| Variables not loaded | Use `terraform.tfvars` |
| Output wrong | Use `.tags.Name` |

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge**:  
Fully private EC2 with VPC-only access

**💡 Solution**:  
```hcl
map_public_ip_on_launch = false
cidr_blocks = [var.KKE_VPC_CIDR]
```

**🎯 Key Success Factors**:  
- ✅ `map_public_ip_on_launch = false`  
- ✅ SG ingress from VPC CIDR only  
- ✅ Variables & outputs with exact names  
- ✅ No internet access

---

## ⚠️ Important Production Notes

### **Next Steps**:
- Add NAT Gateway for outbound  
- Use Private Route Table  
- Enable VPC Flow Logs

### **Security**:
```hcl
# Optional: Restrict to specific ports
ingress {
  from_port = 22
  to_port   = 22
  protocol  = "tcp"
  cidr_blocks = [var.KKE_VPC_CIDR]
}
```

---

## ✅ Task Completion Checklist

- [ ] SSH'd into jump host as `bob`
- [ ] Navigated to `/home/bob/terraform/`
- [ ] Created `main.tf` with all resources
- [ ] Added VPC with correct CIDR variable
- [ ] Added subnet with `map_public_ip_on_launch = false`
- [ ] Created security group with VPC-only ingress
- [ ] Added EC2 instance in private subnet
- [ ] Created `variables.tf` with `KKE_VPC_CIDR` and `KKE_SUBNET_CIDR`
- [ ] Created `terraform.tfvars` with correct values
- [ ] Created `outputs.tf` with required outputs
- [ ] Ran `terraform init` successfully
- [ ] Ran `terraform validate` with success
- [ ] Ran `terraform plan` and reviewed
- [ ] Ran `terraform apply -auto-approve`
- [ ] Verified VPC created with correct CIDR
- [ ] Verified subnet has no public IP assignment
- [ ] Verified security group allows only VPC CIDR
- [ ] Verified EC2 has no public IP
- [ ] Verified EC2 has private IP in subnet range
- [ ] Verified outputs display correctly
- [ ] Documented all steps

**🎉 Success Criteria Met When**:
- Four files exist: `main.tf`, `variables.tf`, `terraform.tfvars`, `outputs.tf`
- VPC `xfusion-priv-vpc` with CIDR `10.0.0.0/16`
- Subnet `xfusion-priv-subnet` with CIDR `10.0.1.0/24`
- Subnet has `map_public_ip_on_launch = false`
- Security group allows ingress only from VPC CIDR
- EC2 instance `xfusion-priv-ec2` in private subnet
- EC2 has no public IP address
- Variables use exact names: `KKE_VPC_CIDR`, `KKE_SUBNET_CIDR`
- Outputs use exact names: `KKE_vpc_name`, `KKE_subnet_name`, `KKE_ec2_private`
- All resources created successfully

---

## 🏁 Task Completion Summary

**Completed**:
- ✅ Private VPC with custom CIDR
- ✅ Private subnet with public IP disabled
- ✅ Security group with VPC-only access
- ✅ EC2 instance in private subnet
- ✅ Variables and outputs properly configured
- ✅ Fully isolated private infrastructure

**Final Status**: Task 98 completed successfully!  
**Outcome**: Private VPC infrastructure created with internal-only access in AWS us-east-1 — ready for validation.

---

## 🎓 Learning Outcomes

- ✅ Creating private subnets
- ✅ Disabling public IP assignment
- ✅ VPC-internal security groups
- ✅ Using Terraform variables
- ✅ Creating `terraform.tfvars` files
- ✅ Defining Terraform outputs
- ✅ Multi-file Terraform projects
- ✅ Private infrastructure patterns

---

## 🚀 Next Steps

- Add NAT Gateway for outbound internet
- Create private route tables
- Implement VPC endpoints
- Add VPC Flow Logs
- Create bastion host for access
- Implement VPN connection
- Add Systems Manager Session Manager

