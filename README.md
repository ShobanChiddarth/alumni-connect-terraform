# alumni-connect-infrastructure

Terraform IaC for deploying the [alumni-connect-backend](https://github.com/ShobanChiddarth/alumni-connect-backend) on AWS.

## Architecture

![architecture](./assets/architecture.png)

## Security groups

| Resource | Inbound | Outbound |
|---|---|---|
| bastion | SSH + ICMP from 0.0.0.0/0 | all |
| backend-server | SSH + ICMP from bastion, 8000 from ALB | all |
| database-server | SSH + ICMP from bastion, 5432 from backend | all |
| ALB | 80 from 0.0.0.0/0 | 8000 to 0.0.0.0/0 |

## Prerequisites

- Terraform >= 1.2
- AWS credentials with EC2, VPC, ELB permissions
- `.ssh/` directory inside `infrastructure/` (bastion private key is written here after apply)

## Deploy
```bash
cd infrastructure
export AWS_ACCESS_KEY_ID="..."
export AWS_SECRET_ACCESS_KEY="..."
terraform init
terraform apply
```

After apply, destroy the NAT gateway to stop incurring charges:
```bash
terraform destroy \
  -target=aws_nat_gateway.alumni-nat-gw \
  -target=aws_eip.nat-gateway-elastic-ip
```

## SSH access
```bash
# Into bastion
ssh -i infrastructure/.ssh/alumni-bastion.pem ubuntu@<bastion-public-ip>

# Into private instances via bastion
ssh -i infrastructure/.ssh/alumni-bastion.pem ubuntu@<bastion-public-ip>
ssh -i ~/.ssh/alumni-management.pem ubuntu@<ec2-private-ip>
```

## Variables

| Variable | Description |
|---|---|
| `db_password` | PostgreSQL password for the `alumni_connect` database |

Pass via `terraform.tfvars` or environment variable `TF_VAR_db_password`.


## Destroy NAT Gateway

```
terraform destroy -target=aws_nat_gateway.alumni-nat-gw -target=aws_eip.nat-gateway-elastic-ip
 ```
 