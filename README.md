# databricks-home-lab
Setup of AWS Customer Managed VPC with ultra cost saving for Databricks


# From My Desk to Databricks: How I Built a Cost-Effective AWS Customer Managed VPC with a NAT Instance for Databricks

## 🌍 Real-Life Spark: Why I Built a CMVPC at Home

It started over a quiet weekend at home.

As a AWS Cloud architect constantly working with data platforms, I realized I needed something more hands-on, more raw than a managed sandbox. I wanted to simulate a **production-like AWS Databricks setup**—but in a **low-cost, fully controlled way**, without the usual enterprise cloud budget.

> Build a minimal, cost-optimized Databricks-ready AWS VPC with full private networking, just like in production.

## 🧩 Why Choose CMVPC for Databricks?

Databricks allows deployment in a **Customer Managed VPC**, where **you control the network**, endpoints, routing, and security.

### Benefits:

- 🔒 Data Governance
- ⚙️ Customization
- 🔄 Monitoring
- 💰 Cost Optimization
- 🧪 Lab Testing

## 💸 Why Not NAT Gateway? The Hidden Cost

| Component         | Monthly Cost |
|------------------|--------------|
| NAT Gateway      | $32          |
| NAT Instance     | $3           |
| Savings/month    | ~$29         |

## 🧱 VPC Architecture Overview

- **Region**: `eu-west-2 (Ireland)`
- **One Availability Zone**
- **Public Subnet**: `10.0.1.0/24`
- **Private Subnet**: `10.0.2.0/24`

### Network Diagram

![AWS CMVPC Databricks NAT Instance Architecture](assets/databricks-cmvpc-architecture.png)

## 🛠️ NAT Instance Setup

```bash
#!/bin/bash
# Enable IP forwarding
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

# Setup NAT with nftables
dnf install -y nftables
systemctl enable --now nftables

cat <<EOF > /etc/nftables.conf
#!/usr/sbin/nft -f

table ip nat {
    chain postrouting {
        type nat hook postrouting priority 100;
        oifname "eth0" masquerade
    }
}
EOF

systemctl restart nftables
```

## 🔁 Verify Internet Access

```bash
curl https://www.google.com
```

## 🔌 Add VPC Endpoints

### Gateway Endpoint

- `com.amazonaws.eu-west-2.s3`

### Interface Endpoints

- logs
- monitoring
- ssm
- ec2messages
- ssmmessages

## 📦 CloudFormation

### EC2 Creation

```yaml
Resources:
  MyPrivateEC2:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro
      ImageId: ami-0abcdef1234567890
      SubnetId: subnet-0abcde1234567890
      KeyName: my-key
```

### Interface Endpoint

```yaml
Resources:
  SSMEndpoint:
    Type: AWS::EC2::VPCEndpoint
    Properties:
      VpcId: vpc-0abcdef1234567890
      VpcEndpointType: Interface
      ServiceName: com.amazonaws.eu-west-2.ssm
      SubnetIds:
        - subnet-0abcde1234567890
      SecurityGroupIds:
        - sg-0abcde1234567890
      PrivateDnsEnabled: true
```

## 🔐 IAM Role

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<databricks-account-id>:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "<databricks-external-id>"
        }
      }
    }
  ]
}
```

## 🔗 Next Step

Subscribe to Databricks on AWS Marketplace and connect to your CMVPC.

## 🧪 Final Thoughts

✅ Real-world testing, ✅ Cost control, ✅ Perfect for labs.

---

