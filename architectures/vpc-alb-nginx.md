## Architecture: ALB + Bastion + Private EC2

### Flow
User -> ALB -> EC2 (Nginx)

### Admin Flow
Admin -> Bastion -> EC2 (SSH)

### Internet Acces
EC2 -> NAT -> Internet

### Notes
- Bastion Host is in Public Server (for Admin/developers to access EC2 via SSH)
- EC2 Instances are in private subnets
- EC2 insatnces are spread over 2 AZ's for high availabilty
- Security Group is attached to each EC2 with http:80 and SSH rules
- Target Group is attached to Load Balancer by linking 2 private EC2 instance to route the requests
- IGW is used to provide Internet to VPC and Public subnet gets the internet via routing through IGW
- NAT used for outbound access




            Internet
                │
                ▼
               IGW
                │
                ▼
        ┌───────────────┐
        │ Public Subnet │
        │               │
User ─▶ ALB             │
        │               │
Admin - -▶ Bastion      │
        └──────┬────────┘
               │
               ▼
        ┌───────────────┐
        │ Private Subnet│
        │               │
        │    EC2        │
        └──────┬────────┘
               │
               ▼
             NAT
               │
               ▼
              IGW
               │
               ▼
            Internet

