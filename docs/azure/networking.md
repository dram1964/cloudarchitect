# Azure Networking

## Virtual Networks

[Microsoft VNet Docs](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview)

Azure Virtual Network (VNet) is a logical isolation of the Azure Cloud dedicated to your 
subscription. Azure VNet is an IaaS resource, providing a software-defined private network. 

- VNets belong to a resource group and can only be part of one Region
- Traffic entering VNet and region is not billed: only traffic leaving the VNet and Region is billed
- External VNet routing is either:
    - Hot-Potato routing: packet is passed to the internet as soon as possible
    - Cold-Potato routing: packets stay on Microsoft network as long as possible. This means packet is handed to an Edge Point of Presence to be delivered to its final destination. Cold-Potato routing ensures the packets spends the least time on the internet, and results in fast delivery and lower latency
- Internet communication - VMs can connect to the internet by default. To connect to the VM from the internet you can assign a Public IP address or put the VM behind a public load balancer
- Communication between Azure resources - some services (VMs, App Services, AKS, VMSS) can communicate via the Virtual Network. Other services (CosmosDB, Service Bus, Key Vault, Storage Accounts, Azure SQL Db) require Service Endpoints
- Address Space - the address space for each VNet needs to be unique in your subscription and for any other network that you wish to connect to
- NAT Gateway - you can configure a subnet to use a static outbound IP address when accessing the internet
- Bastion Host - can be enabled for the VNet
- DDoS Protection - can be enabled for the VNet
- Subnet Delegation - designate a subnet to be used for a dedicated service

Each VNet has its own CIDR address block and can be either isoloted or linked to other 
VNets and on-prem networks as long as the CIDR blocks do not overlap. 

Control traffic into a VNet using a VPN Gateway, an NSG or a firewall

- A firewall can control access to multiple VNets across multiple subscriptions
- A VPN Gateway can only be applied to a single VNet
- NSGs are applied at the Subnet or Device level

The virtual network can be segmented into subnets 
(**use ipcalc to help with subnet addressing**). Azure reserves the first 4 and the last IP 
address from each subnet. For the 192.168.1.0/24 subnet, the reservations are:

1. 192.168.1.0 - network address
2. 192.168.1.1 - default gateway
3. 192.168.1.2 and 192.168.1.3 - azure DNS
4. 192.168.1.255 - broadcast address

Address can be either private (for use within the VNet) or public (for communication over the
internet and with public-facing Azure resources). Public IP addresses can be asssigned to:

- Virtual Machines
- Load Balancers
- VPN Gateways
- Application Gateways

Public IPs come in two SKUs. The Standard SKU only supports static assignment, is only available
for network interfaces, load balancers, application gateways and VPN gateways. Standard SKU provides 
zone-redundancy and is closed to inbound traffic by default (an NSG must be configured to allow
inbound traffic). 

Public IP Address prefixes are a reserved, static range of public IP addresses. Address prefixes
are region-dependant and can be associated to availability zones within the region. 

Private IPs are assigned from the subnet range that the resource is deployed to. Private IPs
support static or dynamic assignment to:

- Virtual Machines
- Load Balancers (front-end config)
- Application Gateways (front-end config)

## Network Security Groups

NSGs are used to limit traffic to resources in a virtual network. An NSG contains rules that
allow or deny inbound or outbound traffic. NSGs can be assigned to one or more subnets or NICs. 
Inbound traffic is evaluated at the subnet first, then at the NIC. This is reversed for outbound
traffic. For the traffic to be allowed, the NSGs must contain an 'allow' rule at both levels. 

The default rules in an NSG:

- Deny all inbound traffic except from the VNet and Azure Load Balancers
- Allow outbound traffic to the VNet and the Internet

NSGs filter traffic to and from Azure resources within a VNet. Rules are defined by specifying:

- Source of traffic
- Source port of traffic
- Destination of traffic
- Destination port of traffic
- Protocol used

Rules are assigned an action (either allow or deny) and a priority to determine the order in 
which they are processed. Higher priority rules override the lower priority rules. NSGs can be 
associated with multiple Subnets and NICs, but a Subnet and NIC can only have one NSG associated.
NSGs can only be associated to resources in the same region and subscription. Firewalls are 
required to filter traffic across multiple subscriptions

- Use <code>az network nsg list</code> command to list NSGs
- Use <code>az network nsg rule list</code> to inspect NSG rules


Application Security Groups logically group Virtual Machines by workload: NICs for 
specific VMs can be assigned to the appropriate ASG. The ASGs can then 
be used as source or destination in an NSG. This allows you 
to control traffic to the servers without needing to specify specific IP addresses. For example, 
an NSG can allow internet inbound to the Web Servers ASG, allow the Web Servers to communicate 
with the Database servers, and deny other traffic to reach the Database servers. 
Members of an ASG must be on the same VNet. When NSG rules refer to two ASGs, they must 
both be on the same VNet. Only 3,000 ASGs allowed per subscription


## [Azure Firewall](https://learn.microsoft.com/en-us/azure/firewall/)

- Uses a static, public IP for your virtual network resources
- Integrated with Azure Monitor to enable logging and analytics
- User-Defined Routing used to control traffic
- Inbound and Outbound Filtering rules
- Application rules - allowed FQDNs that can be accessed from a subnet
- Inbound Destination Network Address Translation (DNAT)
- Outbound Source Network Address Translation (SNAT)
- Normally Deployed to a central VNet to control general network access
- Premium SKU adds:
    - TLS inspection
    - Intrusion Detection Systems
    - Intrusion Prevention Systems
    - URL Filtering
    - Web Categories
    
- Third-Party NVAs are available via the Azure Marketplace as an alternative to Azure Firewall

Azure Firewall is a stateful firewall (analyses connections not just individual packets) 
with high availability and unrestricted scalability. Used 
to protect your VNets across subscriptions. Availability zones are configured during deployment. 

Application rules allow you to restrict traffic using FQDNs. Filtering rules can be specified
with IP addresses, ports and protocols. Threat-intelligence feed from Microsoft can be used
to filter traffic from known malicious sources. By default, Azure Firewall blocks all traffic
unless you enable it. 

A hub-and-spoke model is recommended for implementing Azure Firewall. The Firewall should be 
placed in the hub VNet to filter traffic to and from peers and the on-prem network. 

1. DNAT Rules - DNAT (Destination Network Address Translation) rules are used to translate your Firewall Public IP and port to a private IP and port. NAT rules must be accompanied by a matching network rule that allows the traffic. NAT rules are useful for publishing SSH, RDP or non-HTTP/S applications to the internet. 
2. Network Rules - Specifies the ip addresses, ports and protocols to allow for non-HTTP/S traffic (although http(s) traffic can be targetted using port numbers 80 and 443) 
3. Application Rules - Specify FQDNs or FQDN tags that can be accessed from a subnet. 

DNAT rules are examined first, then Network rules and finally Application rules irrespective 
of the priority assigned to the rules. Rules are organised into Rule Collections and Rule 
Collection Groups. Rules in a rule Collection must be of the same type. Rule Collection Groups
can contain Rule Collections of different type. Rule Collections in a Rule Collection Group and
Rules in a Rule Collection are processed by priority with 100 being the highest priority 
and 65,000 being the lowest. However, the DNAT-Network-Application processing is order is also 
preserved, so the processing order will be:

1. DNAT Rule Collection Group
2. DNAT Rule Collection
3. Network Rule Collection Group
4. Network Rule Collection
5. Application Rule Collection Group
6. Application Rule Collection

Network rules and application rules are terminating: if a matching rule is found, no subsequent
rules are processed. 

If Threat Intelligence filtering is enabled, these rules are highest priority and processed 
before network and application rules. If IDPS is configured in *Alert and Deny* mode, these 
rules are applied after the network and application rules have been processed. 

## VNet Routing

Azure automatically creates system routes and assigns the routes to each subnet in a VNet.
System routes cannot be created or deleted manually, but the can be over-ridden with 
Custom Routes. Each route consists of an address prefix and a next hop. The following
default routes are added to each subnet in a VNet

| Source | Address Prefix | Next Hop | 
|--------|----------------|----------|
|Default | VNet address space | Virtual Network |
|Default | 0.0.0.0/0 | Internet |
|Default | 10.0.0.0/8 | None |
|Default | 172.16.0.0/12 | None |
|Default | 192.168.0.0/16 | None |
|Default | 100.64.0.0/10 | None |


The Virtual Network next-hop rule for the VNet address spaces, ensures that all traffic is 
routed within the VNet. Azure creates a route for each address range defined in the 
VNet address space. Subnets are within the VNet address space and therefore subnet routing 
is also handled by the VNet routes.

The Internet next-hop rule, means that all traffic not covered by the Virtual Network rule, 
is routed via the internet. If the traffic is to an Azure service then this is routed over 
the Azure backbone. 

The None next-hop rules means that all traffic to these addresses are dropped, unless you 
have used these addresses in a VNet, in which case the Virtual Network rule will be used to 
replace the None rule. 

Additional Routes are added when specific Azure features are used:

1. VNet Peering: a route is added for each address space within the VNets of the peering
2. Virtual Network Gateway: when using a Virtual Network Gateway, routes are added to the VNet with a next-hop type of Virtual Network Gateway. The source is also 'Virtual Network Gateway' because the gateway adds the routes to the subnet. The address prefixes are either:
  
    - propogated from on-prem via BGP or
    - configured locally on the local network gateway

3. Virtual Network Service Endpoint: the public IP addresses for certain services are added to the route table when you enable a service endpoint to a service. Service endpoints are enabled for individual subnets within a virtual network, so the route is only added to the subnet the service endpoint belongs to. The public IP addresses of Azure services change periodically and these are automatically updated in the route tables. 

Custom routes can be created to change the default behaviour provided by System Routes. For
instance, when you want to direct traffic to a Virtual Network Applicance such as a Firewall. 
Custom routes can be created in a route table or by exchanging BGP routes with on-prem gateways.
Route Tables can be associated to zero or more VNet subnets. Each subnet can have no more than 
one route table associated. User-defined routes are combined with the default routes, and the 
user-defined routes take precedence. The next-hop types for custom routes are: 

1. Virtual Appliance: a virtual machine running a network applicance, for example a firewall. For Virtual Appliance next-hop rules, you also need to specify an next-hop IP address which can be one of:

    - a private IP address for the appliance
    - a private IP address for an Azure internal load balancer

2. Virtual Network Gateway. The Virtual Network Gateway must be created with type *VPN*. You can not use a Virtual Network Gateway of type *ExpressRoute* in a custom route, as ExpressRoute gateways must use BGP
3. None: drops all traffic for the specified address prefix
4. Virtual Network: when you need to override the routing within a virtual network.
5. Internet: used to explicitly route traffic over the internet, or to route traffic to Azure services with a Public IP via the Azure backbone

You can not use VNet-Peering or VirtualNetworkServiceEndpoint as the next-hop type in 
user-defined routes, as these are created automatically by Azure when VNet-Peerings or Service 
Endpoints are created.

Service Tags can be used to specify address-prefixes in custom routes. Service Tags are groupings
of IP addresses for specific Azure services. Service Tags are automatically updated when the IP 
address prefixes change. Routes with explicit IP-prefixes are given precedence over routes using
service tags. Where more than one service tag has matching IP prefixes, precedence is decided in
this order:

1. Regional Tags
2. Top Level Tags (i.e. service type)
3. AzureCloud Regional Tags
4. AzureCloud Tags

Azure uses the most-specific route if two matching routes existing in the routing table. Otherwise
custom routes are used before BGP routes and BGP routes are chosen before system routes. 

The default system-route for 0.0.0.0/0 routes all traffic over the internet (or the Azure Backbone
for Azure Services) if no other matching rule is encountered. You can override this with a 
custom route that sets the next-hop as either a Virtual Appliance or a Virtual Network Gateway.
This means that traffic to Azure Services will also be routed to the Virtual Appliance or 
Network Gateway unless you enable Service Endpoints for the service. Service Endpoints will 
create routes for the service which will take precedence over the default route. 

Also, if you override the default 0.0.0.0/0 route, then you will no longer be able to directly 
access resources on that subnet from the Internet. Indirect access to the resource is still 
possible if inbound traffic passed through the device specified as the next-hop type for the rule.
If the next-hop type is *Virtual Appliance* then this must:

- Be accessible from the internet
- Have a public IP address
- Not have any NSG rule associated to it that denies connection to the target
- Not deny the connection
- Be able to NAT and forward the traffic or be able to proxy the connection

If the next-hop type is *Express Route*, then your on-prem gateway can NAT-and-forward or proxy 
the connection using ExpressRoute private peering. 

If using a VPN-gateway, don't create a 0.0.0.0/0 custom-route in the Gateway subnet as this 
will prevent the VPN-gateway from working. 

[Routing Example](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#routing-example)

## [Service Endpoints](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview)

Service Endpoints allow you to secure Azure service resources to only your VNets by 
enabling only Private IP addresses in the VNet to reach the endpoint of a service. 
The endpoint extends the VNet identity to the service resource. Endpoints provide access
to services in the same region. 

Service Endpoints are available for:

- AAD
- CosmosDB
- Cognitive Services
- Container Registry
- EventHub
- KeyVault
- ServiceBus
- SQL
- Storage
- Web

Endpoints are enabled on subnets configured in the VNet. For Azure SQL and Data Lake Storage, 
the endpoint only applies to traffic within the same region. Once you enable a service endpoint
in your subnet, you can add a Virtual Network rule to secure the service resources to your VNet 
and deny traffic to the service via the internet. Service Endpoints ensure that traffic from 
the VNet to the service travels on the Azure Backbone only.

By default, Azure services secured by service endpoints will not be reachable from on-premises
devices. You can allow access to the service from Public IPs using the firewall configuration 
for the service.

With Service Endpoints, DNS entries for Azure services continue to the Public IP Addresses 
assigned to the service. 

To secure Azure services to multiple subnets within or across VNets, you can enable service
endpoints on each subnet independantly. If you want to inspect or filter the traffic sent to 
the Azure service, you can apply the service endpoint to the VNA subnet, and then route traffic
from the VNet through the VNA. 

There is no additional charge for using service endpoints and there is no limit on the number
of service endpoints in a virtual network.

## [Private Link](https://learn.microsoft.com/en-us/azure/private-link/private-link-overview?toc=%2Fazure%2Fvirtual-network%2Ftoc.json)

Private Link provides a method to access Azure PAAS, customer-owned or partner services privately 
without using the public internet. Private Link allows private access to services in other Regions.
Private endpoints are accessible using private peering or VPN tunnels from on-prem or peered 
VNets. 

Private Link is used to map the private endpoint to the PAAS service, thus bringing the 
service into your private virtual network. 

Azure PaaS services are normally deployed with a Public IP address. When you access these 
services from an Azure VNet via a Public IP, traffic is routed over the internet.

Private Link allows you to replace the Public IP address for the service with a private 
network interface. The private network interface is a private endpoint which is assigned an 
IP address from the range assigned to your subnet. 

Private Link Services also allows you to create private endpoints for your own custom Azure services. 
The custom service must be placed behind an Azure Standard Load Balancer and the private endpoint is 
attached to the Load Balancer's front-end IP configuration. When you create your own 
Private Link Service it is assigned a unique name (*prefix.guid.suffix*). This name can then be 
used by consumers of the service to establish a private endpoint connection to the service 
in their own VNets. 

Private endpoints are mapped to your Azure virtual networks, so that the services appear to be
part of the VNet. Private Link allows private access even across Regions. All traffic is 
routed using the Azure Backbone and therefore does not pass over the Internet. Once
Private Link is enabled for the service, the public endpoint for the service can be disabled
to prevent any public access. 

Peered virtual networks can also access Private Link resources and on-premise networks can also 
use the Private Link connections via ExpressRoute or VPN Gateway. 

If you configure the private endpoints to integrate with your private DNS zone, then the FQDN is
automatically assigned to the private endpoint. 

Private endpoints are charged per hour and per GB of inbound and outbound traffic. 

## [Azure DNS](https://learn.microsoft.com/en-us/azure/dns/)

When a tenant is created, an initial domain name is assigned in the '.onmicrosoft.com' space. 
You can add a domain name with Azure DNS to a subscription and resource group. Use 
[GoDaddy](https://wwww.godaddy.com/domains/domain-name-search) to search
for un-used domain name. You can then delegate the zone to Azure DNS by setting up
NS records to point to Azure DNS servers. You can then add A records for the public IPs of 
your VMs and use the name servers listed in the NS records to resolve the VM IP addresses. 

To use your own DNS servers, the custom domain name 
must be added to your directory and verified. Verification is performed by adding a 
TXT or MX record to the domain.  Azure will then 
query the domain for this record. If successful, the domain name is then added to the 
subscription and resources can be configured to use this domain. 

Azure DNS provides a secure DNS service to manage and resolve domain names in a virtual 
network. A DNS zone hosts DNS records for a domain: to host your domain in Azure you need 
to create a DNS zone for that domain name. DNS zones are added to resource groups. The DNS 
zone name must be unique in the resource group, but can be used in other resource groups and 
subscriptions. The root/parent domain is registered at the registrar and pointed to Azure DNS.
Child Domains are registered in Azure DNS directly.

You do not need to own the domain name to host the domain in Azure: but you do need to own the
domain to configure the domain.

To delegate your domain to Azure DNS, you need to update the parent domain with the NS records
from your Azure DNS zone. 

Private DNS zones enable you to use your own custom domain names (e.g. contoso.org) 
and provides name resolution for VMs within a virtual network and between virtual networks. 
The Private DNS zone needs to be linked to the VNet(s). When the VNet link is created, 
you can enable 'auto registration' for connected devices. DNS records for the private zone
are not viewable or retrievable, but the records are registered and will resolve. 

For a single VNet, when you create a Private Zone and link it to the VNet, VMs in the VNet can 
carry out dns forward (name to IP) and reverse (IP to name) resolution for the private IP addresses
of the VMs. 

For multiple VNets linked to a common private DNS zone, forward DNS lookups are resolved, but
reverse DNS lookups are only resolved for addresses in the same VNet.

Alias Records can be added to Azure DNS using Azure Resource Names (e.g. the resource 
name of a Load Balancer public IP address). This way, when the resource IP address updates, the
DNS record will also dynamically update. 

## VNet Peering

VNet Peering allows you to connect two or more VNets so that they appear as
a single network from a connectivity perspective. There are two types of peering:
regional for VNets in the same region and; global for VNets in different regions.

VNet Peering keeps traffic on the Azure Backbone and provides a low-latency, 
high-bandwidth connection between resources. 

A VPN Gateway can be used in one of the peered VNets to enable the other peers
to access resources outside the VNet. This would
typically be used in the Hub VNet of a Hub-and-Spoke model to allow the spokes
to communicate with on-prem networks via the Hub VPN Gateway. This setup requires
'Allow Gateway Transit' enabled in the hub VNet and 
'Allow remote gateways' in the spoke VNets.

VNet Peering is non-transitive, but you can configure user-defined routes and
service chaining to provide transitivity. 

## [VPN Gateway](https://learn.microsoft.com/en-us/azure/vpn-gateway/)

See also [Create a Site-to-Site VPN connection](https://learn.microsoft.com/en-us/azure/vpn-gateway/tutorial-site-to-site-portal?source=docs)

A VPN Gateway is used to connect two trusted private networks using an encrypted 
tunnel over a public, untrusted network. A VPN Gateway can be used to connect 
Azure VNets and On-prem networks and typically provides up to 1 Gbps bandwidth. 

Only one VPN Gateway can be deployed per VNet, but multiple connections can be made
to the same VPN Gateway. Connections can be: 

- Site to Site 
- VNet to VNet
- Point to Site (connects an individual device)

For a Site-to-Site connection, the following steps are required to configure
a VPN connection:

1. Create VNets and Subnets

    requires a reservation from the on-site network address range. Ensure that there is no overlap between on-prem and cloud networks/subnets.

2. Specify the DNS server (optional)

    required for name resolution for resources deployed in the VNet

3. Create the Gateway Subnet

    requires a /28 or /27 CIDR block. The Gateway subnet must be called 'GatewaySubnet'

4. Create the VPN Gateway

    when creating the virtual network gateway you can choose between VPN or ExpressRoute as the gateway type, and route-based or policy-based as the VPN type. Policy-based only supports IKEv1, comes with a Basic SKU only, and provides only one tunnel. Policy-based VPNs direct traffic using IPsec policies. Route-based is the usual choice and uses route table to direct traffic. The SKU will determine the number of tunnels available and the aggregate throughput of the connection. The different SKUs provide between 30-100 S2S connections, 250 to 10,000 P2S connections and 650Mbps to 10.0 Gbps aggregate throughput. During this step you will also be able to choose between active-active or active-standby mode.
    
5. Create the Local Network Gateway

    A reference to the On-Prem location. Contains a name for the connection, the IP address or FQDN of the on-prem VPN device, and one or more IP address ranges that define the address range for the on-prem network

6. Configure the On-Prem VPN Device

    Check the [list of validated devices](https://learn.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpn-devices)

7. Create the VPN Connection

    Uses the Local Network Gateway, Virtual Network Gateway and a Pre-Shared Key (PSK). The Shared Key value must match the value used when configuring the on-prem VPN Device in Step 6.

A VPN Gateway consists of two or more VMs deployed to a dedicated Gateway Subnet in the
VNet. VPN Gateways support Availability Zones. When connecting to a single 
on-site VPN, the VPN Gateway can be configured in Active/Standby mode, where only
one instance connects to the on-site VPN device. Failover is automatic and 
should occur in 10 to 15 seconds for planned maintainence or 60 to 90 seconds 
for unplanned outtages. If two on-site VPNs are available then the Azure VPN 
Gateway can be configured in Active/Active mode. 

- A VNet can only have one VPN Gateway and the VPN Gateway can only be associated with one VNet.
- VNet address spaces can not overlap with on-prem network address spaces. 
- Use a pre-shared key for authentication
- Internet Key Exchange (IKE) is used to set up a security association (an agreement of the encryption) between two endpoints
- IPSec suite used to encrypt/decrypt data based on the security association
- VPN is either:
    1. Policy-based
        - Specifies static IP address for encryption through the tunnel
        - supports IKEv1 only
        - relies on static route definitions
        - primarily used where legacy on-prem VPN devices require them
    2. Route-based
        - IPSec tunnel modelled as a network interface or vitual tunnel interface
        - IP routing decides whether to use IPSec tunnel
        - supports IKEv2
        - Can use dynamic routing protocols
        - more resilient to creation of new subnets
- Deploy VPN Gateway
    - GatewaySubnet - add a subnet called <strong>GatewaySubnet</strong> to the VNet with a netmask of at least /27
    - Public IP address - Basic SKU Public IP needed as target for on-prem VPN device
    - Local Network Gateway: defines the on-prem network address spaces that will be connecting to Azure and the on-prem VPN used. The on-prem VPN appliance is defined either by IP Address or FQDN
    - Virtual Network Gateway - routes traffic between the VNet and on-prem network
    - Connection: creates a logical connection between the Local Network Gateway and the VPN Gateway
- VPN gateways can also be used for ExpressRoute failover or in Zone-redundant configuration.

## ExpressRoute and VWAN

Use [Azure ExpressRoute](https://learn.microsoft.com/en-us/azure/expressroute/) to 
create private connections between Azure Data Centres and your
on-prem network: ExpressRoute connections do not go over the public internet. Connection is 
private but not encrypted.
      
ExpressRoute connections can be made through an approved connectivity provider using:

1. Colocation: using an Exchange with Azure co-location
2. Point-to-Point Ethernet connection
3. IPVPN Connection: connect your WAN directly using Multi-Protocol Label Switching (MPLS) VPN

ExpressRoute provides bandwidth up to 100 Gbps and low latency and is a good option for 
heavy-duty data transfers such as storage, backups and recovery. ExpressRoute can also be
used for Hybrid applications such as a Web App in Azure that uses on-prem AD for authentication
and authorisation. 

ExpressRoute Features:

1. Layer 3 connectivity - uses BGP to exchange routes with on-prem routers
2. Redundancy - each ExpressRoute connection consists of two BGP connections to two 
Microsoft Enterprise Edge (MSEE) routers. 
3. Regional Connectivity - the peering location for the ExpressRoute connection grants access
to resources in the same geopolitical region. 
4. Global Connectivity - with ExpressRoute Premium add-on, the connection provides access to 
resources from all regions, except national clouds
5. On-Prem connectivity - on-prem datacenters in different locations can be connected together
using ExpressRoute circuits if ExpressRoute Global Reach is enable. This way traffic between
the two datacenters does not need to traverse the public internet.

ExpressRoute can be used in addition to Site-to-Site VPN: you just need to configure two 
VPN Gateways: one of type VPN and another of type ExpressRoute. 

[Azure Virtual WAN](https://learn.microsoft.com/en-us/azure/virtual-wan/) 
provides a cloud-hosted network hub that provides transitive network 
connectivity to sites that are connected using combinations of Point-to-Site, Site-to-Site and
ExpressRoute connections (spokes). Azure VWAN comes in two SKUs: Basic only supports the use
of Site-to-Site VPN connections; Standard supports all types of VPN connections.

## Network Watcher

Add the network watcher agent to a VM:

    Set-AzVMExtension `
    -ResourceGroupName $rgName `
    -Location $location `
    -VMName $vmName `
    -Name 'networkWatcherAgent' `
    -Publisher 'Microsoft.Azure.NetworkWatcher' `
    -Type 'NetworkWatcherAgentWindows' `
    -TypeHandlerVersion '1.4'

## Load Balancer

Load balancing refers to evenly distrubuting load across a group of backend resources. Azure
Load Balancer works at Layer 4 (Transport), and therefore uses IP address and Port to route traffic. 
The load balancer distributes inbound flows that arrive 
at the front-end to the backend resources according to load-balancing rules and health probes. 
The backend resources can be either VMs or instances in a VMSS. 

Public load balancers are used to load balance internet traffic to your VMs. 
The Load Balancer maps public IP address and port of incoming traffic to private IP address and
port of the VM. Mapping is also provided for the response traffic from the VM (SNAT).

The default distribution mode for Azure Load Balancer is a five-tuple hash (source ip, source 
port, destination IP, destination port and protocol). An alternate distribution mode, 
source IP affinity, is available which uses a two-tuple hash (source IP and destination IP) 
or a three-tuple hash (source IP, destination IP and protocol). 

Internal load balancers are used to load balance traffic inside a VNet or to load balance traffic
to Azure resources via a VPN connection. 

Load Balancers aren't physical instances: Load Balancer objects are used to express how 
Azure configures its infrastructure to meet your requirements.

Availability Sets protect from hardware failures within a datacentre and provide a 99.95% SLA.
Availability Zones protect from entire datacentre failure and provide a 99.99% SLA. Availability
Zones provide redundancy across datacentres in the same Region. 

Load Balancers come in two main SKUs:

- Basic: supports 300 instances in backend pools, HTTP and TCP health probes, no zone redundancy,
NSGs are optional, no SLA
- Standard: supports 1000 instances in the backend pool, HTTPS, HTTP and TCP health probes, zone-redundant and zonal frontends for inbound and outbound traffic, inbound flows closed by default unless allowed via an NSG, 99.99% SLA

For the Basic SKU, backend pool devices are limited to machines in a single availability set or VMSS.
Standard SKU can use any VM in a single VNet, availability set or VMSS.

Load balancer rules map a given frontend IP:Port to a set of backend IP:Port combinations: before
configuring the rule, you should create the frontend, backend and health probe. Session
persistence can be based on:

- None: (default) any VM can handle the request
- Client IP: successive requests from the same IP are handled by the same VM
- Client IP and Protocol: successive requests from the same IP:Port combination are handled by the same VM

HTTP health probes rely on a 200 Status response from the backend within 31 seconds. TCP probes
expect a successful connection on a specified port: you can configure Port, Interval and Unhealthy
Threshold. Guest agent probes can also be used, but only when HTTP/TCP probes are not possible.

Traffic Manager is a DNS-based load balancer, that provides load-balancing at the Global 
(cross-regional) level. 

## Application Gateway

Azure Application Gateway implements load-balancing at Layer 7 (Application), and makes 
decisions based on attributes of the HTTP request such as the URI path (path-based routing) 
or address (multi-site routing).
The backend pool can include VMs, VMSS, App Service and even on-prem servers. Application 
Gateway uses round-robin to load balance requests and provides the following features:

- Support for HTTP, HTTP/2, HTTPS, and Websockets
- Application firewall: checks requests for common security threats
- End-to-end request encryption
- Autoscaling
- Redirection
- Rewrite HTTP Headers
- Custom Error Pages

Application Gateway health probes can accept HTTP responses between 200 and 399 to indicate OK. If
no health probe is configured, App Gateway configures a default the expects a response within 30
seconds. 

Application Gateway can load balance within a Region: for Global Load Balancing at the 
Application Layer use Azure Front Door. 

## Azure Content Delivery Network
- Includes Web Application Firewall (WAF) service
  
## Azure DDoS Protection

- Discards DDoS traffic at the Azure network edge  
- Protects against over-consumption of resources
- A singel DDoS protection plan is enabled at the tenant level and used across multiple subscriptions
- Default Basic plan is free
- Standard SKU adds protection from:
    - volumetric attacks
    - protocol attacks
    - resource layer (application) attacks

## Azure Traffic Manager
- a DNS-Based traffic load balancer
- lets you distribute traffic across global Azure regions
  
## CLI Commands

    az network route-table list -o table
    az network route-table list --query "[].[name,resourceGroup,routes[].[addressPrefix,nextHopType, nextHopIpAddress]]

    az network nic show-effective-route-table -n $NIC -g $RESOURCE_GROUP

## Resticting Access to PaaS Resource with Service Endpoints

    RG_NAME=rg-play-tut001
    VNET_NAME=vnet-play-tut001
    LOCATION=uksouth
    PUBLIC_SNET=snet-public
    PRIVATE_SNET=snet-private
    PRIVATE_NSG=nsg-private
    STORAGE_NAME=stgplaytut001
    FILE_SHARE=fs-play-tut001

1. Create the VNET with Public Subnet

        az group create -n $RG_NAME --location $LOCATION

        az network vnet create \
        --name $VNET_NAME \
        --resource-group $RG_NAME \
        --address-prefix 10.0.0.0/16 \
        --subnet-name $PUBLIC_SNET \
        --subnet-prefix 10.0.0.0/24
  
2. List the Services supporting Service Endpoints

        az network vnet list-endpoint-services --location $LOCATION -o table

3. Create an Additional Subnet with a Storage Service Endpoint

        az network vnet subnet create \
        --vnet-name $VNET_NAME \
        --resource-group $RG_NAME \
        --name $PRIVATE_SNET \
        --address-prefix 10.0.1.0/24 \
        --service-endpoints Microsoft.Storage
  
4. Restrict Access for the Private Subnet

        az network nsg create \
        --resource-group $RG_NAME \
        --name $PRIVATE_NSG
        
        # add an NSG to the Private Subnet
        az network vnet subnet update \
        --vnet-name $VNET_NAME \
        --name $PRIVATE_SNET \
        --resource-group $RG_NAME \
        --network-security-group $PRIVATE_NSG
 
        # allow outbound access to Storage Public IPs
        az network nsg rule create \
        --resource-group $RG_NAME \
        --nsg-name $PRIVATE_NSG \
        --name Allow-Storage-All \
        --access Allow \
        --protocol "*" \
        --direction Outbound \
        --priority 100 \
        --source-address-prefix "VirtualNetwork" \
        --source-port-range "*" \
        --destination-address-prefix "Storage" \
        --destination-port-range "*"
        
        # deny outbound to all public IP Addresses
        az network nsg rule create \
        --resource-group $RG_NAME \
        --nsg-name $PRIVATE_NSG \
        --name Deny-Internet-All \
        --access Deny \
        --protocol "*" \
        --direction Outbound \
        --priority 110 \
        --source-address-prefix "VirtualNetwork" \
        --source-port-range "*" \
        --destination-address-prefix "Internet" \
        --destination-port-range "*"
        
        # allow SSH inbound
        az network nsg rule create \
        --resource-group $RG_NAME \
        --nsg-name $PRIVATE_NSG \
        --name Allow-SSH-All \
        --access Allow \
        --protocol Tcp \
        --direction Inbound \
        --priority 120 \
        --source-address-prefix "*" \
        --source-port-range "*" \
        --destination-address-prefix "VirtualNetwork" \
        --destination-port-range "22"
  
5. Create a Storage Account
  
        az storage account create \
        --name $STORAGE_NAME \
        --resource-group $RG_NAME \
        --sku Standard_LRS \
        --kind StorageV2
  
        saConnectionString=$(az storage account show-connection-string \
        --name $STORAGE_NAME \
        --resource-group $RG_NAME \
        --query 'connectionString' \
        --out tsv)
  
6. Create a File Share in the Storage

        az storage share create \
        --name $FILE_SHARE \
        --quota 2048 \
        --connection-string $saConnectionString > /dev/null
  
7. Deny all access to the Storage Account

        az storage account update \
        --name $STORAGE_NAME \
        --resource-group $RG_NAME \
        --default-action Deny
  
8. Allow access to the Storage from the Private Network

        az storage account network-rule add \
        --resource-group $RG_NAME \
        --account-name $STORAGE_NAME \
        --vnet-name $VNET_NAME \
        --subnet $PRIVATE_SNET
  
9. Create VMs on both Subnets

        az vm create \
        --resource-group $RG_NAME \
        --name myVmPublic \
        --image UbuntuLTS \
        --vnet-name $VNET_NAME \
        --subnet $PUBLIC_SNET \
        --generate-ssh-keys
        
        az vm create \
        --resource-group $RG_NAME \
        --name myVmPrivate \
        --image UbuntuLTS \
        --vnet-name $VNET_NAME \
        --subnet $PRIVATE_SNET \
        --generate-ssh-keys
  
10. SSH to VMs and mount the Share (only succeeds on privateVM)

        ssh <publicIpAddress>

        sudo mkdir /mnt/MyAzureFileShare
        sudo mount --types cifs //$STORAGE_NAME.file.core.windows.net/$FILE_SHARE /mnt/MyAzureFileShare --options vers=3.0,username=$STORAGE_NAME,password=$AccountKey,dir_mode=0777,file_mode=0777,serverino

11. Try to list share from own device (fails)

        az storage share list \
        --account-name $STORAGE_NAME \
        --account-key $AccountKey

