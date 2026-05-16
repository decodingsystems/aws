# Lab 10: VPC Peering Between Two VPCs

## Objective
Connect two isolated VPCs so their resources can communicate privately, without traffic going over the public Internet.

## Scenario
- VPC A (`10.0.0.0/16`) — your application VPC
- VPC B (`10.1.0.0/16`) — your database/services VPC

## Steps

### Part 1: Create Two VPCs
**VPC A:**
1. VPC → Create VPC → CIDR: `10.0.0.0/16`, Name: `VPC-A`
2. Create subnet `10.0.1.0/24` in VPC-A, Name: `SubnetA`
3. Create Internet Gateway, attach to VPC-A
4. Update route table for SubnetA: `0.0.0.0/0 → IGW`

**VPC B:**
1. Create VPC → CIDR: `10.1.0.0/16`, Name: `VPC-B`
2. Create subnet `10.1.1.0/24`, Name: `SubnetB`

### Part 2: Create VPC Peering Connection
1. VPC → **Peering connections** → Create peering connection
   - Name: `VPC-A-to-VPC-B`
   - Requester: `VPC-A`
   - Accepter: `VPC-B` (same account, select from dropdown)
   - Create.
2. A pending request appears. Select it → **Actions → Accept request**.

### Part 3: Update Route Tables (BOTH sides)
**VPC-A route table:**
- Add route: `10.1.0.0/16 → Peering Connection (pcx-xxx)`

**VPC-B route table:**
- Add route: `10.0.0.0/16 → Peering Connection (pcx-xxx)`

### Part 4: Test Communication
1. Launch EC2 in VPC-A SubnetA (with public IP, SG allows SSH + ICMP from anywhere for testing).
2. Launch EC2 in VPC-B SubnetB (no public IP, SG allows ICMP from `10.0.0.0/16`).
3. SSH into EC2-A. Now ping EC2-B using its **private IP**:
   ```bash
   ping 10.1.1.x   # EC2-B private IP — succeeds!
   ```
   Traffic flows through the peering connection, never touches the Internet.

### Key Limitations of VPC Peering
- CIDRs must not overlap.
- NOT transitive: if A peered with B, and B peered with C, A cannot reach C through B.

## Cleanup
Terminate instances → Delete peering connection → Delete subnets, IGW, route tables, VPCs.
