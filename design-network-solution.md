# Design a network solution

- https://docs.microsoft.com/en-us/learn/paths/architect-network-infrastructure/

## VPN Gateways

- IPSec tunnel
- Azure VPN gateways enable the following connectivity:
  - _Site-to-site_ connections
  - _Point-to-site_ connections
  - _Network-to-network_ connections
- VPN type:
  - Policy-based
    - Support for IKEv1 only
    - Use static routing
    - Used for specific scenarios, like compatibility with legacy VPN devices
  - Route-based
    - Preferred method connection method, since more resilient to topology changes
- Gateway sizes

| SKU       | Site-to-site / VNet-to-VNet tunnels | Throughput | BGP suppor    |
| --------- | ----------------------------------- | ---------- | ------------- |
| Basic     | Max 10                              | 100Mbps    | Not supported |
| VpnGw1/Az | Max 30                              | 650 Mbps   | Supported     |
| VpnGw2/Az | Max 30                              | 1 Gbps     | Supported     |
| VpnGw3/Az | Max 30                              | 1.25 Gbps  | Supported     |

- What's required to deploy a VPN gateway
  - VNet with a subnet
  - Gateway subnet
  - Public IP
  - Local network gateway
  - VPN gateway

![diagram](https://docs.microsoft.com/en-us/learn/modules/connect-on-premises-network-with-vpn-gateway/media/2-resource-requirements-for-vpn-gateway.svg)

- High availability
  - Active/Standby
  - Active/Active
  - ExpressRoute failover
  - Zone-redundant deploy

Network created in the excercise

![excercise networks](https://docs.microsoft.com/en-us/learn/modules/connect-on-premises-network-with-vpn-gateway/media/4-resources-deployed-during-exercise-final.svg)

## ExpressRoute

- Extend on-premise network into MS cloud (Azure, O365, D365) with private high-bandwidth connection
- Communication doesn't go thru public internet
- More reliable, minimal latency, higher througput

- Architecture:

![ExpressRoute](https://docs.microsoft.com/en-us/learn/modules/connect-on-premises-network-with-expressroute/media/3-azure-expressroute-overview.svg)

- To implement ExpressRoute, you need to work with an ExpressRoute partner
- VNet can be connected to an ExpressRoute circuit with a VPN gateway

- Peering schemes

![peering schemes](https://docs.microsoft.com/en-us/learn/modules/connect-on-premises-network-with-expressroute/media/3-azure-peering.svg)

- High availability

  - Each ExpressRoute circuit has two connections different MS edge routers
  - Can set up multiple ExpressRoute circuits, even across multiple providers

- Benefits
  - Predictable performance
  - Data privacy
  - High-througput (up to 10 Gbps, with ExpressRoute Direct up to 100 Gbps), low-latency
  - 99,95 SLA

## Network security

- Network security groups
  - Assigned to a network interface or subnet
  - Consists of rules to allow/deny traffic
  - Service tags
    - Simplify security config
    - MS manages, can't create your own
    - E.g. VirtualNetwor, AzureLoadBalancer, Internet, AzureTrafficManager, Storage, SQL, AppService
  - Application security groups
    - Can group VMs logically and specify their network security rules
    - E.g. web servers (ports 80, 8080) and DB servers (port 1433)

![Application security groups](https://docs.microsoft.com/en-us/learn/modules/secure-and-isolate-with-nsg-and-service-endpoints/media/2-asg-nsg.svg)

- Virtual network service endpoints
  - Connect Azure resources (such as storage) directly to VNET private address space
  - Traffic remains in Azure (doesn't go thru internet)
  - Available in many services, such as Storage, SQL DB, Cosmos DB, Key Vault, Service Bus, Data Lake
  - To enable service endpoint:
    - 1. Turn off public access to service
    - 2. Add service endpoint to a VNET

![VNET service endpoints](https://docs.microsoft.com/en-us/learn/modules/secure-and-isolate-with-nsg-and-service-endpoints/media/4-service-endpoint.svg)

- Service endpoints and hybrid networks
  - Services with service endpoints are not by default accessible from on-premises networks (e.g. with ExpressRoute)
  - NAT IPs need to be added to the service's firewall config

![Service endpoint hybrid](https://docs.microsoft.com/en-us/learn/modules/secure-and-isolate-with-nsg-and-service-endpoints/media/4-service-endpoint-flow.svg)

## VNET peering

- Two VNETs can be peered so they can communicate as if they were on the same network.
- Traffic goes only thru Azure backbone
- Types of peering:
  - VNET peering: within same region
  - Global VNET peering: in different regions
- Have to be created in both VNETs to work
- Can even peer VNETs across subscriptions
- If A and B are peered, B and C are peered, A and C can't communicate
- IP addresses can't overlap

## Azure Traffic Manager

- DNS based traffic load balancer
- Distributes traffic to different regions
- Applications are configured as endpoints:
  - Azure endpoints - services hosted in Azure
  - External endpoints - IPv4/IPv6, FQDNs or services hosted outside Azure
  - Nested endpoints – Combine Traffic Manager profiles
- Routing methods:
  - Weighted routing - Randomly choose an endpoint based on given weights ![Weighted](https://docs.microsoft.com/en-us/learn/modules/distribute-load-with-traffic-manager/media/2-weighted.svg)
  - Performance routing - Choose an endpoint based on lowest latency for the user ![Performance](https://docs.microsoft.com/en-us/learn/modules/distribute-load-with-traffic-manager/media/2-performance.svg)
  - Geographic routing - Choose an endpoint based on where DNS query originates ![Geographic](https://docs.microsoft.com/en-us/learn/modules/distribute-load-with-traffic-manager/media/2-geographic.svg)
  - Multivalue routing - Return multiple healthy endpoints in a single DNS query
  - Subnet routing - Map IP ranges to specific endpoints
  - Priority routing - Prioritized list of endpoints ![Priority](https://docs.microsoft.com/en-us/learn/modules/distribute-load-with-traffic-manager/media/2-priority.svg)

## Azure Load Balancer

- Distribute traffic across multiple VMs
- Two distribution modes:
  - Hash-based distribution (5 tuple):
    - Source IP & port
    - Destination IP & port
    - Protocol
  - Source IP affinity (2 or 3 tuple):
    - Source IP
    - Destination IP
    - (Protocol)

![Load balancer](https://docs.microsoft.com/en-us/learn/modules/improve-app-scalability-resiliency-with-load-balancer/media/2-load-balancer-distribution.svg)

- For high availability deploy VMs to
  - Availability set (99,95% SLA)
  - Availability zone (99,99% SLA)
- Two tiers:
  - Basic
    - Port forwarding
    - Automatic reconfiguration
    - Health probes
    - Outbound connections through source network address translation (SNAT)
    - Diagnostics through Azure Log Analytics for public-facing load balancers
  - Standard
    - All that basic has
      - HTTPS health probes
      - Availability zones
      - Diagnostics through Azure Monitor, for multidimensional metrics
      - High availability (HA) ports
      - Outbound rules
      - A guaranteed SLA (99.99% for two or more virtual machines)
- External load balancer permits traffic from the internet
- Internal load balancer permits traffic from Azure resources

## Application Gateway

- Routes requests based on request URL (_application layer routing_)
- Target can be VMs, VMSS, App Service, on-prem services

![Application Gateway](https://docs.microsoft.com/en-us/learn/modules/load-balance-web-traffic-with-application-gateway/media/2-application-gateway.svg)

- Two types of routing rules:
  - Path based - E.g. `/images/*` and `/video/*`
  - Multiple site hosting - E.g. contoso.com and fabrikam.com
- Other features:

  - Redirection, rewrite HTTP headers, Custom error pages
  - Automatic round-robin load balancing based on host names and paths
  - Web app firewall
  - Autoscaling

- Components & Config
  ![WAF components](https://docs.microsoft.com/en-us/learn/modules/load-balance-web-traffic-with-application-gateway/media/4-application-gateway-components.svg)

  - Front-end IP - Can have more than one, can be public, private or both
  - Listeners - Receive incoming requests and route to back-end pool. Can be _Basic_ or _Multi-site_.
  - Routing rules - Binds a listener to back-end pools. Protocol, Session stickiness, Connection draining, Request timeout, Health probes
  - Back-end pools - Collection of servers. Each pool has its own LB.

- Web application firewall

  - Checks requests for OWASP threats, such as:
    - SQL injections
    - Cross-site scripting
    - Command injection
    - etc.
  - Only available in WAF tier

- Tiers:
  - _Standard_
  - _WAF_
  - Both have V1 and V2. V2 supports availability zones
- Sizes:
  - Small, Medium, Large

## VNET routing

- System routes
  - Are present in all VNET subnets. Can't be deleted, but can be overriden
  - Contains default routes:
    - Allow access to everywhere within the VNET
    - Allow access to Internet (0.0.0.0/0)
    - Block:
      - 10.0.0.0/8
      - 172.16.0.0/12
      - 192.168.0.0/16
      - 100.64.0.0/10
  - ![System routes](https://docs.microsoft.com/en-us/learn/modules/control-network-traffic-flow-with-routes/media/2-system-routes-subnets-internet.svg)
  - In addition there are other system routes, created when enabled:
    - VNET peering
    - Service chaining
    - VNET gateway
    - VNET service endpoint
  - Service chaining:
    - Can override peering rules by creating user-defined routes between peered networks
    - ![Service chaining](https://docs.microsoft.com/en-us/learn/modules/control-network-traffic-flow-with-routes/media/2-virtual-network-peering-udrs.svg)
- Custom routes
  - 2 types: UDR and BGP
  - User-defined route (UDR)
    - E.g. route traffic through firewalls or Network Virtual Appliance (NVA)
    - Next hop type can be:
      - Virtual appliance – E.g. firewall. Defined by the IP address
      - Virtual network gateway – VPN
      - Virtual network
      - Internet
      - None
  - Border Gateway Protocol (BGP)
    - Typically used with ExpressRoute
- Route selection and priority
  - More specific routes have higher prio (/24 is before /16)
  - Routes with same address prefix, order is:
    - UDR
    - BGP
    - System routes
- Network Virtual Appliance (NVA)
  - VM that controls flow of network traffic, often in a perimeter network (e.g. DMZ)
  - E.g. firewall, WAN optimizer, router, load balancer, proxy

## Designing an IP addressing scheme

- 3 ranges of IPs that won't be routed over internet:
  - 10.0.0.0/8
  - 172.16.0.0/12
  - 192.168.0.0/16
- Azure VNETs use these blocks
  - Smallest subnet is /29, largest /8
- In Azure, you typically:
  - use subnets to isolate frontend and backend services
  - use NSG to filter internal and external traffic at network layer
  - firewall has more extensive features for network and application layer filtering
- On-prem and Azure IP addresses can't overlap on interconnected networks
- The first 3 IPs and last IP are reserved for all subnets.

- In Azure there are two types of IP addresses

  - Both of these can be either static or dynamic

  - 1. Public IP addresses

    - Can be assigned to a VM, LB, VPN gateway, App gateway
    - Can't bring own public IP
    - Can't be moved between regions
    - Can create IP address prefix --> Get a contiguous block of IPs
    - Dynamic IPs
      - The default allocation method
      - Can change
      - Is allocated when VM starts, deallocated when it stops
      - Each region has a pool of public addresses
    - Static IPs
      - Will not change during its lifespan
    - Two SKUs:
      - Basic
        - 4-30min adjustable inbound originated flow idle timeout
        - 4min fixed outbound flow idle timeout
        - Open by default. Need NSG to restrict traffic
        - Don't support availability zones
      - Standard
        - 4-30min adjustable inbound originated flow idle timeout
        - 4min fixed outbound flow idle timeout
        - Secure by default. Must explicitly allow inbound traffic
        - Are zone-redundant by default

  - 2. Private IP addresses
    - Can be dynamic (DHCP lease) or static (DHCP reservation)

- Gather requirements:
  - How many devices do you have on the network?
  - How many devices are you planning to add to the network in the future?
  - For expanding:
    - Based on the services running on the infrastructure, what devices do you need to separate?
    - How many subnets do you need?
    - How many devices per subnet will you have?
    - How many devices are you planning to add to the subnets in future?
    - Are all subnets going to be the same size?
    - How many subnets do you want or plan to add in future?

## Hybrid networks

- Azure VPN Gateway

  - Site-to-site
    - ![site2site](https://docs.microsoft.com/en-us/learn/modules/design-a-hybrid-network-architecture/media/4-s2s-connection.svg)
  - Point-to-site
    - ![point2site](https://docs.microsoft.com/en-us/learn/modules/design-a-hybrid-network-architecture/media/4-p2s-connection.svg)
  - Benefits:
    - Well-known, easy to configure and maintain
    - All data encrypted
    - Better suited for light data-traffic
  - Considerations:
    - Uses the internet
    - Potential latency issues
    - Max bandwidth of 1.25 Gbps
    - For site-to-site, you need local VPN device

- ExpressRoute with VPN failover

  - ![expressroutewithfailover](https://docs.microsoft.com/en-us/learn/modules/design-a-hybrid-network-architecture/media/4-expressroute-vpn-failover-architecture.svg)
  - Benefits:
    - Resilient, high-available networks
  - Considerations:
    - With failover, bandwidth is reduced
    - ExpressRoute and VPN gateway must be in the same VNET
    - Highly complex to configure
    - Requires both ExpressRoute and VPN connections
    - Requires redundant VPN gateway and local VPN hardware

- Hub-spoke network topology

  - Single VNET as a hub, connected to on-prem
  - Spokes are other VNETs peered to the hub
  - ![hub-spoke](https://docs.microsoft.com/en-us/learn/modules/design-a-hybrid-network-architecture/media/4-hub-spoke-architecture.svg)
  - Benefits:
    - Sharing and centralized services on the hub can reduce duplication in spokes
    - Subscription limits are overcome by peering
    - Allows separation of organizational work ares into spokes
  - Considerations:
    - What are shared on the hub and what remains on the spokes

- VPN Gateway vs ExpressRoute

| Capability             | VPN Gateway                            | ExpressRoute              |
| ---------------------- | -------------------------------------- | ------------------------- |
| Azure services support | Azure Cloud Services and               |                           | Azure Virtual Machines | Microsoft Cloud Platform |
| Bandwidth              | Up to 10 Gbps                          | Up to 10 Gbps or 100 Gbps | (direct) |
| Protocol               | SSTP or IPsec                          | Direct over VLAN or MPLS  |
| Routing                | Static or dynamic                      | Border Gateway Protocol   | (BGP) |
| Connection resiliency  | Active-passive or                      |                           | active-active | Active-passive or active-active |
| Use case               | Prototyping, dev, test, labs, RDC, and |                           | small production workloads | Access to all Azure | services, enterprise-grade, supporting critical large-scale workloads |
| SLA                    | 99.95-99.99%                           | 99.95%                    |

## Monitor and troubleshoot network infra

- Azure Network Watcher
  - Diagnose the health of Azure networks
  - **Monitoring tools**
    - Topology
      - Generates a graph of the network
      - ![Topology tool](https://docs.microsoft.com/en-us/learn/modules/troubleshoot-azure-network-infrastructure/media/2-network-watcher-topology.png)
    - Connection Monitor
      - Check and monitor connections between resources
      - Measure latency
    - Network Performance Monitor
      - Track and alert latency and packet drops
      - Monitor endpoint-to-endpoint connectivity:
        - Between branches and datacenters
        - Between VNETs
        - Between on-prem and cloud
        - ExpressRoute
  - **Diagnostics tools**
    - IP flow verify
      - Tells if packets are allowed or denied for a specific VM. 5-tuple packet.
    - Next hop
      - Check how packet gets from VM to any destination
    - Security group view
      - Displays all NSG rules appliedto a NIC
    - Packet capture
      - Record all packets sent to and from a VM
      - Is a VM extension
    - Connection troubleshoot
      - Check TCP connectivity between a source and destination VM
      - Shows
        - Latency in ms
        - Number of proble packets sent
        - Numbers of hops in the route
    - VPN troubleshoot
      - Diagnose virtual network gateway connections
  - **Usage and quotas**
    - Shows usage and quotas for
      - Network interfaces
      - NSGs
      - VNETs
      - Public IPs
    - ![UsageQuotas](https://docs.microsoft.com/en-us/learn/modules/troubleshoot-azure-network-infrastructure/media/4-usage-and-quotas.png)
  - **Logs**
    - Flow logs
      - Show ingress and egress IP traffic on NSGs
      - Show 5-tuple and whether was allowed or denied
    - Diagnostic logs
      - Enable/disable logs for network resources
    - Traffic analytics
      - Investigate user and app activity
      - Analyzes NSG flow logs
      - See open ports, traffic flow patterns etc.
