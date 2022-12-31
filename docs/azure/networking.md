# Azure Networking

## Virtual Networks

[Microsoft VNet Docs](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview)

Azure Virtual Network is a logical isolation of the Azure Cloud dedicated to your subscription. 
Each VNet has its own CIDR address block and can be linked to other VNets and on-prem networks
as long as the CIDR blocks do not overlap. The virtual network can be segmented into subnets 
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
for network interfaces and load balancers, provides zone-redundancy and is closed to inbound
traffic by default. 

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

Application Security Groups logically group Virtual Machines by workload: NICs for 
specific VMs can be assigned to the appropriate ASG. The ASGs can then 
be used as source or destination in an NSG. This allows you 
to control traffic to the servers without needing to specify specific IP addresses. For example, 
an NSG can allow internet inbound to the Web Servers ASG, allow the Web Servers to communicate 
with the Database servers, and deny other traffic to reach the Database servers. 

## [Azure Firewall](https://learn.microsoft.com/en-us/azure/firewall/)

Azure Firewall is a stateful firewall with high availability and unrestricted scalability. Used 
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

## VNet Default Routing

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

## Custom Routes

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

## Azure Routing Decisions

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

## VNet [Service Endpoints](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview)

Service Endpoints allow you to secure Azure service resources to only your VNets by 
enabling only Private IP addresses in the VNet to reach the endpoint of a service, without requiring
a Public IP address on the VNet. The endpoint extends the VNet identity to the service resource. 
Connections are still made through the services public endpoint (over the internet). 

Endpoints are enabled on subnets configured in the VNet. For Azure SQL and Data Lake Storage, 
the endpoint only applies to traffic within the same region. Once you enable a service endpoint
in your subnet, you can add a Virtual Network rule to secure the service resources to your VNet 
and deny traffic to the service via the internet.

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

Private Link provides a method to access Azure services privately without using the Internet. 

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

## VNet Peering

VNet Peering allows you to connect two or more VNets so that they appear as
a single network from a connectivity perspective. There are two types of peering:
regional for VNets in the same region and; global for VNets in different regions.

VNet Peering keeps traffic on the Azure Backbone and provides a low-latency, 
high-bandwidth connection between resources. 

A VPN Gateway can be used in one of the peered VNets to enable the other peers
to access resources outside the VNet by allowing Gateway Transit. This would
typically be used in the Hub VNet of a Hub-and-Spoke model to allow the spokes
to communicate with on-prem networks via the Hub VPN Gateway. 

VNet Peering is non-transitive, but you can configure user-defined routes and
service chaining to provide transitivity. 

## [VPN Gateway](https://learn.microsoft.com/en-us/azure/vpn-gateway/)

A VPN Gateway can be used to connect Azure VNets and On-prem networks. Only one
VPN Gateway can be deployed per VNet, but multiple connections can be made
to the same VPN Gateway. Connections can be: 

- Site to Site
- VNet to VNet
- Point to Site (connects an individual device)

A VPN Gateway consists of two or more VMs deployed to a Gateway Subnet in the
VNet. VPN Gateways support Availability Zones. 

For a Site-to-Site connection, the following steps are required to configure
a VPN connection:

1. Create VNets and Subnets

    requires a reservation from the on-site network address range

2. Specify the DNS server (optional)

    required for name resolution for resources deployed in the VNet

3. Create the Gateway Subnet

    requires a /28 or /27 CIDR block. The Gateway subnet must be called 'GatewaySubnet'

4. Create the VPN Gateway

    when creating the virtual network gateway you can choose between VPN or ExpressRoute as the gateway type, and route-based or policy-based as the VPN type. Policy-based only supports IKEv1, comes with a Basic SKU only, and provides only one tunnel. Policy-based VPNs direct traffic using IPsec policies. Route-based is the usual choice and uses route table to direct traffic. The SKU will determine the number of tunnels available and the aggregate throughput of the connection. The different SKUs provide between 30-100 S2S connections, 250 to 10,000 P2S connections and 650Mbps to 10.0 Gbps aggregate throughput. 
    
5. Create the Local Network Gateway

    A reference to the On-Prem location. Contains a name for the connection, the IP address or FQDN of the on-prem VPN device, and one or more IP address ranges that define the address range for the on-prem network

6. Configure the On-Prem VPN Device

    Check the [list of validated devices](https://learn.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpn-devices)

7. Create the VPN Connection

    Uses the Local Network Gateway, Virtual Network Gateway and a Shared Key (PSK)

Every Azure VPN Gateway consists of two instances. When connecting to a single 
on-site VPN, the VPN Gateway can be configured in Active/Standby mode, where only
one instance connects to the on-site VPN device. Failover is automatic and 
should occur in 10 to 15 seconds for planned maintainence or 60 to 90 seconds 
for unplanned outtages. If two on-site VPNs are available then the Azure VPN 
Gateway can be configured in Active/Active mode. 

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

Public load balancers are used to load balance internet
traffic to your VMs. The load balancer can also be used to load balance outbound traffic from 
VMs using SNAT (using the Public IP of the load balancer for outbound connections). 

Internal load balancers are used to load balance traffic inside a VNet. 

Traffic Manager is a DNS-based load balancer, that provides load-balancing at the Global 
(cross-regional) level. 

## Application Gateway

Azure Application Gateway implements load-balancing at Layer 7 (Application), and makes 
decisions based on attributes of the HTTP request such as the URI path or request headers. 
Application Gateway can load balance within a Region: for Global Load Balancing at the 
Application Layer use Azure Front Door. 

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

