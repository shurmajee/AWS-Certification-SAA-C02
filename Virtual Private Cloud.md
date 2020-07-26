You may create 5 VPCs per region. 200 subnets allowed per VPC. **VPCs in a region can span across multiple AZs** 

**RFC1918 range for CIDR blocks:**

* 10.0.0.0 - 10.255.255.255 (10.0.0.0/8)
* 172.16.0.0 - 172.31.255.255 (172.16.0.0/12)
* 192.168.0.0 - 192.168.255.255 (192.168.0.0/16)

If a VPC needs to be connected to another network over VPN or to another VPC, the network ranges should be different. 
One can also set a secondary CIDR block for VPC but the block must come from the same address range as primary but they must not overlap. E.g. if you specify primary CIDR as 192.168.0.0/16, you can not create secondary. One may create upto 5 secondary IP ranges for secondary.Smallest vpc allowed is /28 and biggest is /16

* Subnet is a logical container for EC2 instances. Subnet’s CIDR block must be a subset of VPCs CIDR. 
* AWS reserves first four and last ip addresses in each subnet. Obviously subnet cidrs can not overlap. It is possible for a subnet and vpc to have the same CIDR but then you can create only one subnet. 
* Subnet’s dont have secondary CIDRs
* Subnets are linked to availability zones.
* An elastic network interface is a logical networking component in a VPC that represents a virtual network card.

### VPC Networking Components: ###
**Internet gateway:** It allows an instance to connect to the internet. GET data as well as receive requests. Only one IG can be associated with a given VPC. IG does not have an IP address or management console, but it has a resource ID. To use IG one needs to create a default route in the route table that points to IG as a target. 

**Route Tables:** VPC implements IP routing as a software function. A route table must have at least one subnet association. A subnet can not exist without a route table association. In case secondary CIDRs are enabled for VPC one would see multiple local entries in the main route table. Main route tables control routing for subnets without explicit RT associations.
Adding Secondary CIDR Blocks to VPC


**Routes:** Routing decisions are based on destination. Route has two components. 
	* Destination: IP prefix with CIDR notation
	* Target: AWS network resource such as IG or ENI. it can not be CIDR.

When the target is set to local it is a local route (VPC CIDR) which is a mandatory route in every route table. It allows communication between instances in the same VPC. 
If only a local route is added, any other request to other ip prefixes would be dropped. Local route can not be edited. 

To enable internet access one must add a default route pointing to the Internet Gateway. The prefix 0.0.0.0/0 encompasses all ipv4 addresses. 

Any subnet associated with a route table containing a default route pointing to an IG is called a **public subnet**. 
A private subnet does not have a default route pointing to IG. Private subnet may have a default route pointing to a NAT gateway if internet access is needed.

### VPC with a public and private subnet

Default VPC is open to the internet by default. When we create our own vpc and subnets inside, they are private by default. 
We need to add IG and make them public on our own.
The IG and Elastic IPs assigned to each instance in the public subnet ensure that the instances are accessible over the internet. The NAT gateway takes care of the instances in the private subnet. They get internet access but do not get exposed/accessible from outside.

Security groups act like firewalls. They should be the first point of troubleshooting. By default SGs have an allow all outbound rule. SGs control traffic at Instance level. NACL is associated with the entire subnet. By default NACL allows all in/out traffic.

Security Groups: It controls traffic to an from an instance by controlling the ENI of the instance. An ENI can have multiple SGs attached to it. Also a given SG can be attached to multiple ENIs. As most instances have one ENI. The SGs are associated with instances. Multiple instances across subnets can belong to one security group.
Inbound rules: SOURCE | PROTOCOL | PORT RANGE
Outbound rules: DESTINATION | PROTOCOL | PORT RANGE

SRC or DEST in a rule can be a CIDR or resource id of another SG. If we specify a security group as source or destination any instance with the security group would be allowed access.  

When you specify a security group as the source for a rule, traffic is allowed from the network interfaces that are associated with the source security group for the specified protocol and port. For an example, see Default Security Group for Your VPC. Adding a security group as a source does not add rules from the source security group.

Stateful access: SGs act like stateful firewall. E.g. if they allow inward traffic they would also allow corresponding outward traffic.

Elastic IP: Public IP that you request for. Available for use until you release it. Use this as instances may get restarted by AWS during maintenance and get new IPs. To ensure efficient use of Elastic IP addresses, AWS imposes a small hourly charge when they aren't associated with a running instance, or when they are associated with a stopped instance or an unattached network interface. While your instance is running, you aren't charged for one Elastic IP address associated with the instance, but you are charged for any additional Elastic IP addresses associated with the instance.

Network ACL: Network ACLs work at a subnet level (instead of ENI) and over rule the security groups. Network ACLs are stateless and security groups are stateful. ACL works at subnet level security groups work as instance levels. Stateless = default deny. E.g. if acl allows inbound rdp, it also has to allow corresponding outbound port, by default it would deny traffic. With security groups only one rule would work as it is stateful.

A subnet can have only one NACL associated with it. A NACL can be associated with multiple subnets as long as they are in the same VPC. 
You may want to use NACL along with security groups, as it would apply by default to all new instances launched in the subnet. 
In case of application clients that send requests using http but expect a TCP/UDP response on a different random port it is difficult to define an outbound rule in NACL as it is stateless. For such cases do not restrict outbound traffic using NACL.

Rules are evaluated starting with the lowest numbered rule. As soon as a rule matches traffic, it's applied regardless of any higher-numbered rule that might contradict it. *  rule number indicates the last rule in the list. This is a default rule that can not be changed or deleted. This rule denies all traffic that has not been explicitly allowed by the preceding rules.

In practice, to cover the different types of clients that might initiate traffic to public-facing instances in your VPC, you can open ephemeral ports 1024-65535. However, you can also add rules to the ACL to deny traffic on any malicious ports within that range. Ensure that you place the deny rules earlier in the table than the allow rules that open the wide range of ephemeral ports.

One use case for NACL is to specifically block an IP range. Explicit blocking can not be done with a security group. 

Network Address Translation:
NAT devices must always be present in a public subnet as they have to connect to the internet. NAT gateway is a NAT device managed by AWS. nothing to manage or access. Automatically scales to accommodate bandwidth requirements. NAT gateway must be assigned an EIP while creation as it is created in a public subnet.

NAT instance is a preconfigured EC2 instance that uses a linux based AMI. NO autoscaling. Advantage is you may use it as a bastion host. 
NAT gateways are not supported for IPv6 traffic—use an outbound-only (egress-only) internet gateway instead.

If you have resources in multiple Availability Zones and they share one NAT gateway, and if the NAT gateway’s AZ is down, resources in the other Availability Zones lose internet access. To create an Availability Zone-independent architecture, create a NAT gateway in each Availability Zone and configure your routing to ensure that resources use the NAT gateway in the same Availability Zone.

VPC peering: peering interface can be created to communicate between two vpcs. Works over different regions and accounts. It is a point to point connection between two VPCs. It only allows instance to instance communication. You can not use it to share IG or NAT (Transitive routing is not allowed through peering). You may share network load balancers and EFS mount points. No overlapping CIDRs allowed. For VPC peering connection, the desired subnet’s routing table needs to have an entry for the destination VPC CIDR mapped with Peering connection ID (both ways)

VPC transit gateway: A transit gateway is a network transit hub that you can use to interconnect your virtual private clouds (VPC) and on-premises networks. Multiple VPCs can be connected to the gateway.

VPC endpoint enables you to privately connect your VPC to supported AWS services and VPC endpoint services powered by AWS PrivateLink without requiring an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection. Instances in your VPC do not require public IP addresses to communicate with resources in the service. Traffic between your VPC and the other service does not leave the Amazon network. Two types: Interface endpoints/Gateway endpoints

Interface endpoints: An interface endpoint is an elastic network interface with a private IP address from the IP address range of your subnet that serves as an entry point for traffic destined to a supported service.
Pricing: You will be billed for each hour that your VPC endpoint remains provisioned in each Availability Zone, irrespective of the state of its association with the service (0.01$/hour). Data processing charges apply for each Gigabyte processed through the VPC endpoint regardless of the traffic’s source or destination (0.01$/gb).Many services are supported by this. API gateway, cloudformation, SNS, KMS, EC2API, ELB API etc.

Gateway Endpoints: gateway endpoints support only two services currently. S3 and dynamodb. There are no additional costs for gateway endpoints.
Once You create an endpoint to a supported AWS service, and associate your route table with the endpoint. An endpoint route is automatically added to the route table.
You cannot explicitly add, modify, or delete an endpoint route in your route table by using the route table APIs, or by using the Route Tables page in the Amazon VPC console. You can only add an endpoint route by associating a route table with an endpoint.
The endpoint route shows a prefix list ID which represents a list of IP addresses.  See examples here: Gateway VPC endpoints - Amazon Virtual Private Cloud

Gateway VPC endpoints work only within the region. DNS resolution must be enabled in the vpc for them to work. Endpoint connections cannot be extended out of a VPC. Resources on the other side of a VPN connection, VPC peering connection, transit gateway, AWS Direct Connect connection, or ClassicLink connection in your VPC cannot use the endpoint to communicate with resources in the endpoint service.
https://docs.aws.amazon.com/vpc/latest/userguide/vpce-gateway.html#vpc-endpoints-limitations

* **FLOW LOGS** 
Flow logs allow to capture IP traffic information, in and out of network interfaces. They can be created for VPC, subnet, network interface. The contain source and dest ip addresses along with other info. These logs can be delivered to S3 or cloudwatch.

* **Classic link:**
With EC2-Classic, your instances run in a single, flat network that you share with other customers. This was the original release of EC2. ClassicLink allows you to link EC2-Classic instances to a VPC in your account, within the same Region. 
There are two steps to linking an EC2-Classic instance to a VPC using ClassicLink. First, enable the VPC for ClassicLink (this is disabled by default). Then link any running EC2-Classic instance in the same Region in your account to that VPC. This includes selecting security groups from the VPC to associate with your EC2-Classic instance. After you've linked the instance, it can communicate with instances in your VPC using their private IP addresses, provided the VPC security groups allow it.

