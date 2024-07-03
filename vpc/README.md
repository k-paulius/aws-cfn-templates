# Amazon VPC Related AWS CloudFormation Templates

## `vpc-public-private-dual-stack.yaml`

### Description

This template deploys a VPC with a public subnet and one or two private subnets across up to three Availability Zones. This setup supports 2-Tier and 3-Tier network architectures with potential resilience across multiple Availability Zones.

- VPC
    - A `/16` IPv4 CIDR block will be assigned to the VPC, with the second octet derived from the `pIPv4Block` parameter: `10.{pIPv4Block}.0.0/16`.
    - Amazon-provided `/56` IPv6 CIDR block will be assigned to the VPC.
- Subnets:
    - A public subnet and up to two private subnets will be created in each AZ, totaling up to 9 subnets.
    - Each subnet will be assigned a `/20` IPv4 CIDR range.
    - Each subnet will be assigned a `/64` IPv6 CIDR range.
    - Public subnets will be named `{pVPCName}-sn-public-{a/b/c}`.
    - First tier private subnets will be named `{pVPCName}-sn-private-{p1stPrivateSubnetTierName}-{a/b/c}`.
    - Second tier private subnets will be named `{pVPCName}-sn-private-{p2ndPrivateSubnetTierName}-{a/b/c}`.
- Routing
    - A separate routing table will be created for each subnet.
    - An Internet Gateway will be created and attached to the VPC.
    - Routing tables associated with public subnets will have routes for `0.0.0.0/0` and `::/0` directed to the Internet Gateway.
    - Routing tables associated with private subnets will only allow routing within the VPC.
- Network ACLs
    - Two network ACLs will be created `{pVPCName}-nacl-public` and `{pVPCName}-nacl-private`.
    - The public NACL will be associated with all public subnets, and the private NACL will be associated with private subnets from both tiers.
    - Both NACLs allow all outbound traffic.
    - Both NACLs allow all ingress traffic, **except for TCP traffic on port 22 (SSH) and port 3389 (RDP)**.
        - **You must add explicit allow rules to pass SSH and/or RDP traffic.**
- Security Groups
    - Default Security Group **does not allow outbound traffic** and allows inbound traffic from all resources that are assigned to the same SG.
    - Security Group named `{pVPCName}-sg-no-ingress-egress` does not allow inbound or outbound traffic.

### Parameters

- `pVPCName`
    - Name of the VPC
- `pIPv4Block`
    - Second Octet of the VPC network (10.xxx.0.0/16)
- `pNumberOfAZs`
    - Number of Availability Zones (AZs) to use
    - Allowed values: 1, 2, 3
- `pEnable2ndPrivateSubnetTier`
    - Enable a second tier of private subnets, adding an additional private subnet in each Availability Zone (AZ)
    - Allowed values: true, false
- `p1stPrivateSubnetTierName`
    - Name of the first private subnet tier (e.g., t1, web)
    - Default value: t1
- `p2ndPrivateSubnetTierName`
    - Name of the second private subnet tier (e.g., t2, db)
    - Default value: t2
- `pWorkloadIdTag`
    - Value used in the "workload-id" tag
- `pEnvironmentIdTag`
    - Value used in the "environment-id" tag
- `pOwnerNameTag`
    - Value used in the "owner" tag

## `vpc-nat-gateway.yaml`

### Description

This template deploys NAT Gateway(s) into a VPC created using the `vpc-public-private-dual-stack.yaml` template. You can control the number of Availability Zones where the NAT Gateway will be deployed using the `pNumberOfAZs` parameter.

A route for `0.0.0.0/0` directed to the NAT Gateway will only be added to the first tier private subnets.

When single AZ routing is enabled (`pEnableSingleAZRouting=true`), only one NAT Gateway will be created. Traffic from all subnets, as determined by the `pNumberOfAZs` parameter, will be routed through that single NAT Gateway. This is mostly useful for testing purposes, as it sacrifices high availability and incurs charges for cross-AZ traffic.

### Parameters

- `pParentVPCStackName`
    - Name of the parent VPC stack that was created using `vpc-public-private-dual-stack.yaml` template
- `pNumberOfAZs`
    - Number of Availability Zones (AZs) to use
    - *Must not exceed the number of AZs VPC is deployed to*
    - Allowed values: 1, 2, 3
- `pEnableSingleAZRouting`
    - Route traffic from all subnets (AZs) through a single NAT Gateway
    - Allowed values: true, false
- `pWorkloadIdTag`
    - Value used in the "workload-id" tag
- `pEnvironmentIdTag`
    - Value used in the "environment-id" tag
- `pOwnerNameTag`
    - Value used in the "owner" tag

## `vpc-flow-logs-s3.yaml`

### Description

This template creates a VPC flow log that sends traffic flow information to an Amazon S3 bucket for a VPC created using the `vpc-public-private-dual-stack.yaml` template.

### Parameters

- `pParentVPCStackName`
    - Name of the parent VPC stack that was created using `vpc-public-private-dual-stack.yaml` template
- `pFlowLogsBucketName`
    - Name of the Amazon S3 bucket where traffic flow information will be stored
- `pTrafficType`
    - The type of traffic to monitor (accepted traffic, rejected traffic, or all traffic)
    - Allowed values: ACCEPT, REJECT, ALL
- `pWorkloadIdTag`
    - Value used in the "workload-id" tag
- `pEnvironmentIdTag`
    - Value used in the "environment-id" tag
- `pOwnerNameTag`
    - Value used in the "owner" tag
