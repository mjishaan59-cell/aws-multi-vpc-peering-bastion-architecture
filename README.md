\# AWS Multi-VPC Peering \& Bastion Host Architecture



A beginner-friendly AWS networking project demonstrating how to connect two isolated Virtual Private Clouds (VPCs) using VPC Peering, public Bastion Hosts, private EC2 instances, route tables, and Security Groups.



\## 1. Project Overview



This project demonstrates how to build a secure network architecture across two AWS VPCs:



\- \*\*Development VPC:\*\* `10.0.0.0/16`

\- \*\*Production VPC:\*\* `20.0.0.0/16`



Each VPC contains:

\- One public subnet.

\- One private subnet.

\- One public Bastion Host.

\- One private EC2 backend instance.



The two VPCs communicate through an AWS VPC Peering Connection.



\### Project Objectives



1\. Create two separate VPCs with non-overlapping CIDR ranges.

2\. Configure public and private subnets.

3\. Attach Internet Gateways to both VPCs.

4\. Configure public and private route tables.

5\. Establish a VPC Peering Connection.

6\. Configure secure SSH access through Bastion Hosts.

7\. Configure Security Groups for controlled network communication.

8\. Verify private cross-VPC connectivity using ping.



\## 2. Architecture



\### Network Design



| Resource | Development VPC | Production VPC |

|---|---|---|

| VPC CIDR | 10.0.0.0/16 | 20.0.0.0/16 |

| Public Subnet | 10.0.1.0/24 | 20.0.1.0/24 |

| Private Subnet | 10.0.2.0/24 | 20.0.2.0/24 |

| Internet Gateway | dev-igw | prod-igw |

| Public Route Table | dev-public-rt | prod-public-rt |

| Private Route Table | dev-private-rt | prod-private-rt |

| Bastion Host | dev-bastion | prod-bastion |

| Private Instance | dev-private | prod-private |



\### Architecture Diagram



```text

&#x20;                LOCAL COMPUTER

&#x20;                      |

&#x20;                   SSH

&#x20;                      |

&#x20;            +---------+---------+

&#x20;            |                   |

&#x20;       DEV BASTION         PROD BASTION

&#x20;            |                   |

&#x20;      DEV PUBLIC           PROD PUBLIC

&#x20;         SUBNET                SUBNET

&#x20;            |                   |

&#x20;      DEV PRIVATE          PROD PRIVATE

&#x20;         SUBNET                SUBNET

&#x20;            |                   |

&#x20;            +--------+----------+

&#x20;                     |

&#x20;                VPC PEERING

&#x20;                     |

&#x20;               PRIVATE TRAFFIC



&#x20;      DEV VPC                 PROD VPC

&#x20;     10.0.0.0/16             20.0.0.0/16

```



Note: VPC Peering provides private communication between the VPCs. It does not automatically make every resource accessible; route tables and Security Groups must explicitly permit the required traffic.



\## 3. Prerequisites



Before starting, make sure you have:



\- An AWS account.

\- Access to the AWS Management Console.

\- Basic knowledge of VPCs, subnets and EC2.

\- An EC2 key pair.

\- SSH client installed on your local computer.

\- Basic understanding of Linux commands.



For this project, Ubuntu EC2 instances are used. If you choose another Linux distribution, use its appropriate SSH username.



\## 4. Step-by-Step Implementation



\### Step 1: Create the VPCs



Open the AWS VPC Console.



Create the Development VPC:



\- Name: `dev-vpc`

\- IPv4 CIDR: `10.0.0.0/16`



Create the Production VPC:



\- Name: `prod-vpc`

\- IPv4 CIDR: `20.0.0.0/16`



Both VPCs must have non-overlapping CIDR ranges.



\### Step 2: Create Subnets



Create two subnets in each VPC.



\*\*Development VPC\*\*



| Subnet | CIDR |

|---|---|

| dev-public-subnet | 10.0.1.0/24 |

| dev-private-subnet | 10.0.2.0/24 |



\*\*Production VPC\*\*



| Subnet | CIDR |

|---|---|

| prod-public-subnet | 20.0.1.0/24 |

| prod-private-subnet | 20.0.2.0/24 |



\### Step 3: Create and Attach Internet Gateways



Create two Internet Gateways:



\- `dev-igw`

\- `prod-igw`



Attach each Internet Gateway to its corresponding VPC.



\### Step 4: Configure Route Tables



Create four route tables.



\*\*Development VPC\*\*



`dev-public-rt`:



| Destination | Target |

|---|---|

| 10.0.0.0/16 | Local |

| 0.0.0.0/0 | dev-igw |



Associate this route table with `dev-public-subnet`.



`dev-private-rt`:



| Destination | Target |

|---|---|

| 10.0.0.0/16 | Local |



Associate this route table with `dev-private-subnet`.



\*\*Production VPC\*\*



`prod-public-rt`:



| Destination | Target |

|---|---|

| 20.0.0.0/16 | Local |

| 0.0.0.0/0 | prod-igw |



Associate it with `prod-public-subnet`.



`prod-private-rt`:



| Destination | Target |

|---|---|

| 20.0.0.0/16 | Local |



Associate it with `prod-private-subnet`.



\### Step 5: Create VPC Peering Connection



Open VPC Console → Peering Connections → Create Peering Connection.



Configure:



\- Name: `dev-to-prod-peering`

\- Requester VPC: `dev-vpc`

\- Accepter VPC: `prod-vpc`

\- Account: My Account

\- Region: Same Region



Create the connection and accept the peering request.



Wait until its status becomes `Active`.



\### Step 6: Configure Private Route Tables for Peering



Now add routes to allow communication between the two private networks.



\*\*Development Private Route Table\*\*



| Destination | Target |

|---|---|

| 20.0.0.0/16 | VPC Peering Connection |



\*\*Production Private Route Table\*\*



| Destination | Target |

|---|---|

| 10.0.0.0/16 | VPC Peering Connection |



Save the changes.



Important: The peering routes must be configured on both sides. VPC Peering does not automatically configure route tables.



\### Step 7: Launch EC2 Instances



Launch four EC2 instances:



| Instance | Subnet | Public IP |

|---|---|---|

| dev-bastion | dev-public-subnet | Yes |

| dev-private | dev-private-subnet | No |

| prod-bastion | prod-public-subnet | Yes |

| prod-private | prod-private-subnet | No |



Use the same or appropriately configured SSH key pair for the instances.



Make sure public subnet instances receive public IPv4 addresses and private instances do not.



\### Step 8: Configure Security Groups



\*\*A. Bastion Security Groups\*\*



Create:



\- `dev-bastion-sg`

\- `prod-bastion-sg`



Inbound rule:



| Type | Port | Source |

|---|---|---|

| SSH | 22 | Your IP address (/32) |



\*\*B. Private Backend Security Groups\*\*



Create:



\- `dev-private-sg`

\- `prod-private-sg`



SSH access:



| Security Group | Type | Port | Source |

|---|---|---|---|

| dev-private-sg | SSH | 22 | dev-bastion-sg |

| prod-private-sg | SSH | 22 | prod-bastion-sg |



ICMP access for cross-VPC testing:



| Security Group | Type | Source |

|---|---|---|

| dev-private-sg | All ICMP - IPv4 | 20.0.0.0/16 |

| prod-private-sg | All ICMP - IPv4 | 10.0.0.0/16 |



This restricts ICMP traffic to the peered VPC instead of allowing it from the entire internet.



Security Groups are stateful, so response traffic for permitted connections is automatically allowed.



\## 5. SSH Connection and Cross-VPC Testing



\### Important Security Note



Never upload or copy your private SSH key (`.pem`) to a Bastion Host unnecessarily.



Instead, use SSH ProxyJump. This allows your local computer to authenticate through the Bastion Host without placing your private key on the remote server.



\### Step A: Connect to the Development Private Instance



From your local terminal, run:



```bash

ssh -i "key.pem" -J ubuntu@<DEV\_PUBLIC\_BASTION\_IP> ubuntu@10.0.2.210

```



Replace `<DEV\_PUBLIC\_BASTION\_IP>` with your actual development Bastion Host public IP.



You should now be connected to the private Development EC2 instance.



\### Step B: Test Cross-VPC Connectivity



From the development private instance, run:



```bash

ping -c 4 20.0.2.98

```



Here, `20.0.2.98` is the example private IP of the Production EC2 instance.



Expected output:



```text

PING 20.0.2.98 (20.0.2.98) 56(84) bytes of data.



64 bytes from 20.0.2.98: icmp\_seq=1 ttl=64 time=1.24 ms

64 bytes from 20.0.2.98: icmp\_seq=2 ttl=64 time=0.98 ms

64 bytes from 20.0.2.98: icmp\_seq=3 ttl=64 time=1.02 ms

64 bytes from 20.0.2.98: icmp\_seq=4 ttl=64 time=1.02 ms



\--- 20.0.2.98 ping statistics ---

4 packets transmitted, 4 received, 0% packet loss

```



Your actual latency and output will vary.



\### Step C: Verify the Network Route



From the development private instance:



```bash

ip route

```



Check connectivity:



```bash

ping -c 4 20.0.2.98

```



You can also check the private instance's network configuration:



```bash

ip addr

```



If the ping fails, verify the route tables, Security Groups, NACLs and operating-system firewall rules.



\## 6. Troubleshooting



| Problem | What to check |

|---|---|

| Cannot SSH into Bastion | Public IP, Internet Gateway, route table and SSH Security Group |

| Cannot SSH into private instance | Bastion Security Group reference, key pair and SSH username |

| VPC Peering not working | Peering status and routes on both sides |

| Ping fails | ICMP rules, route tables and OS firewall |

| Private instance cannot access internet | Expected without NAT Gateway or another egress solution |

| Cannot connect using ProxyJump | Local key path, SSH username and Bastion connectivity |



Remember that VPC Peering is not transitive. A third VPC cannot automatically use one peering connection to reach another VPC.



\## 7. Security Best Practices



\- Never expose private backend instances directly to the internet.

\- Never commit `.pem` files or other private credentials to GitHub.

\- Restrict SSH access to your own public IP.

\- Allow only required traffic in Security Groups.

\- Avoid using `0.0.0.0/0` for private backend access.

\- Use separate Security Groups for Bastion Hosts and backend instances.

\- Keep private instances in private subnets.

\- Consider AWS Systems Manager Session Manager as an alternative to Bastion Hosts.

\- Enable VPC Flow Logs for network traffic monitoring.

\- Review IAM permissions and avoid unnecessary administrative access.



\## 8. Architecture Limitations



This is a learning and demonstration architecture, not a complete production-ready deployment.



For production environments, consider:



\- Multi-AZ subnet deployment.

\- High availability for Bastion Hosts or Systems Manager access.

\- AWS Systems Manager Session Manager.

\- NAT Gateway where private outbound internet access is required.

\- VPC Flow Logs and CloudWatch monitoring.

\- AWS Network Firewall or other centralized security controls where required.

\- Automated deployment using Terraform or CloudFormation.



\## 9. Project Cleanup



To avoid unnecessary AWS charges, delete resources when the project is no longer needed.



Recommended cleanup order:



1\. Terminate the four EC2 instances.

2\. Delete the VPC Peering Connection.

3\. Delete custom route tables.

4\. Detach and delete Internet Gateways.

5\. Delete Security Groups.

6\. Delete subnets.

7\. Delete the two VPCs.



Check for any remaining resources, such as EBS volumes, Elastic IPs and NAT Gateways, which may continue to incur charges.



\## 10. Skills Demonstrated



By completing this project, you demonstrate practical knowledge of:



\- Amazon VPC networking.

\- Public and private subnets.

\- Internet Gateways.

\- Route table configuration.

\- VPC Peering.

\- EC2 deployment.

\- Bastion Host architecture.

\- Linux SSH administration.

\- Security Groups and network access control.

\- Private network connectivity.

\- Network troubleshooting.



\## 11. Conclusion



This project demonstrates how two separate AWS VPCs can communicate securely through VPC Peering while maintaining controlled access to private EC2 instances.



It is a practical starting point for understanding AWS networking, cloud infrastructure and network security.



\*\*Technologies:\*\* AWS VPC, EC2, Internet Gateway, VPC Peering, Route Tables, Security Groups, Ubuntu Linux and SSH.



\---



\*This project was configured through the AWS Management Console for hands-on learning and demonstration purposes.\*

