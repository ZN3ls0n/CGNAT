# CGNAT
Simplified application of CGNAT

# Physical Topology Details
<ul>
  <li>2x Cisco 2960 (24-port) Switches</li>
  <li>4x PCs</li>
  <li>2x Cisco 2901 Routers</li>
  <li>1x ISR 4331 Router</li>
  <li>1x ISR 4321 Router</li>
  <li>1x Server</li>
</ul>

# Logical Topology Details
<ul>
  <li>2 Customer Private Subnets</li>
    <ul>
      <li>Customer 1: 192.168.10.0/24</li>
      <li>Customer 2: 192.168.20.0/24</li>
    </ul>
  <li>Customer to CGNAT Gateway Subnets</li>
    <ul>
      <li>100.64.10.0/30</li>
      <li>100.64.10.4/30</li>
    </ul>
  <li>CGNAT to ISP Subnet</li>
    <ul>
      <li>203.0.113.0/30</li>
    </ul>
  <li>Internet-Facing Subnet (Web Server)</li>
    <ul>
      <li>198.51.100.0/30</li>
      <li>198.51.100.2/30 (Public Web Server)</li>
    </ul>
</ul>

# Demo Steps
## LAN Design
<ul>
  <li>2 Customer Subnets</li>
    <ul>
      <li>Customer 1: 192.168.10.0/24</li>
      <li>Customer 2: 192.168.20.0/24</li>
    </ul>
  <li>Each customer subnet has:</li>
    <ul>
      <li>Has 2 PCs</li>
      <li>One access switch</li>
      <li>One access router (default gateway)</li>
    </ul>

## IP Addressing Scheme
<ul>
 <li><b>NOTE: </b>LAN Routers have been configured as DHCP server to provide endpoints in the LANs with IP addresses. For a walkthrough on configuring DHCP on a Cisco router, see this <a href="https://nodeconnect.blogspot.com/2025/04/how-to-set-up-cisco-router-as-dhcp.html">post</a>. </li>
 <li>If you choose static configurations, ensure each PC is configured with: </li>
  <ul>
    <li>Valid IP address and subnet mask</li>
    <li>Router default gateway for LAN traffic</li>
  </ul>
</ul>

<b>Troubleshooting</b>
<ul>
  <li>Verify ports are enabled and links are correct</li>
  <li>Verify each PC has the correct IP address, subnet mask, and default gateway</li>
  <li>Ping the default gateway or another PC in the subnet to verify connectivity</li>
</ul>
</ul>

## CGNAT Design
<p>Once LAN configuration is verified, traffic can be routed to the CGNAT gateway. At this point, traffic has only reached the default gateway. Outgoing traffic will have a private source IP address, which will need to be translated to access public-facing resources.</p>
<ul>
  <li>Each customer router has a point-to-point link with the CGNAT gateway.</li>
    <ul>
      <li>Customer 1 CGNAT Link: 100.64.10.0/30</li>
        <ul>
          <li>100.64.10.1 - Customer Router</li>
          <li>100.64.10.2 - CGNAT Gateway</li>
        </ul>
      <li>Customer 2 CGNAT Link: 100.64.10.4/30</li>
      <ul>
        <li>100.64.10.5 – Customer Router</li>
        <li>100.64.10.6 – CGNAT Gateway</li>
      </ul>
    </ul>
</ul>
  <p>These subnets represent traffic from the customer side toward the CGNAT gateway. In many home network deployments, customers do not have a dedicated public IP connection to their ISP. ISPs use the shared space within the 100.64.0.0/10 reserved address space for CGNAT. To preserve space for other customers, the point-to-point links use a /30 subnet. LAN traffic from each customer router is forwarded over its designated CGNAT link.  </p>
  
<p>This is where the NAT process begins. At this stage, the CGNAT router is configured to translate private customer traffic into public IP addresses before routing it to the ISP. Customer traffic is then trunked over the ISP link. Traffic is differentiated using random ephemeral port numbers, a form of port address translation (PAT).</p>


## CGNAT Configuration
**NOTE** This topology uses static routing only. No dynamic routing protocols are configured.

Routing configurations will be different depending on the router's position:
<ul>
  <li>Customer routers require a default route to reach the CGNAT gateway and the Internet. </li>
  <li>The CGNAT gateway requires routes back to the customers and a default route to the Internet.</li>
  <li>The ISP router represents a provider's network, including autonomous systems and protocols used to route traffic across the Internet.</li>
</ul>

### Customer Router Configuration
Each customer router uses a default route pointing to its directly connected CGNAT gateway interface. Before it reaches the CGNAT gateway, the customer router will NAT the internal LAN addresses to a non-private, dedicated subnet range for CGNAT: 100.64.10.0/30. <b>This is to show why CGNAT disrupts many services. It requires explicitly saying which inbound traffic it will accept. On a ISP level, that is not only out of our hands but up to the ISP to manage those connections. Workarounds include port forwarding, a public IP address, or a dual stack configuration. </b>
<ul>
<li>Customer 1 Router:
  <pre><code>
    conf t
    int g0/0
    ip nat inside # LAN traffic
    int g0/1
    ip nat outside # LAN -> CGNAT traffic
    exit
    access-list 1 permit 192.168.10.0 0.0.0.255
    ip nat inside source list 1 int g0/1 overload
    ip route 0.0.0.0 0.0.0.0 100.64.10.2
    exit</code></pre>
</li>
<li>Customer 2 Router: 
  <pre><code>
    conf t
    int g0/0
    ip nat inside # LAN traffic
    int g0/1
    ip nat outside # LAN -> CGNAT traffic
    exit
    access-list 2 permit 192.168.20.0 0.0.0.255
    ip nat inside source list 2 int g0/1 overload
    ip route 0.0.0.0 0.0.0.0 100.64.10.6
    exit</code></pre>
</ul>

### CGNAT Gateway Configuration
The CGNAT gateway handles the inside and outside NAT configuration. The inside NAT will translate the CGNAT traffic to a public address, which will be routed through the ISP before it reaches the Internet. The customer-facing interfaces will have inside configurations and the ISP-facing interface will have outside configurations. 

<pre><code>
  conf t
  int g0/0/0
  ip nat inside
  exit
  int g0/0/1
  ip nat inside
  exit
  int g0/0/2
  ip nat outside
  exit
  ip nat inside source list 1 int g0/0/2 overload
  access-list 1 permit 100.64.10.0 0.0.0.7 # This line allows local traffic from either customer to be NATed using the CGNAT range. This will create a stateful connection which will allow for return traffic. To see stateful connectionse, use <code>show ip nat translations</code>.

  # Static routes for traffic
  ip route 192.168.10.0 255.255.255.0 100.64.10.1
  ip route 192.168.20.0 255.255.255.0 100.64.10.5
  ip route 0.0.0.0 0.0.0.0 203.0.113.2

  exit
</code></pre>

### ISP Router Configuration
Once the customer traffic has reached the CGNAT gateway, it is forwarded to the ISP router. This router allows customer traffic to route across the Internet to reach public-facing servers, as shown in this demo. 

In this topology, the ISP router connects two directly attached networks:

- The CGNAT-to-ISP link: 203.0.113.0/30

- The public web server network: 198.51.100.0/30

Since these two networks are directly connected, you do not need additional static route configurations. This section explains how traffic is handled once it leaves the CGNAT gateway:
-  On the CGNAT gateway, traffic is forwarded to the public web server through the next-hop interface.
  <pre><code>ip route 0.0.0.0 0.0.0.0 203.0.113.2</code></pre>
- When it reaches the server, it will use that interface as its default gateway.
- The ISP router does not utilize NAT because there is no need for a private-to-public IP translation. At this point traffic from the ISP router has a public IP address, so it is routable on the Internet. 

### Public Web Server Configuration

The public web server is assigned a routable IP address within the internet-facing subnet and uses the ISP router as its default gateway.

IP address: 198.51.100.2

Subnet mask: 255.255.255.252

Default gateway: 198.51.100.1

With this configuration in place, the server can respond to traffic originating from customer networks after it has been translated through CGNAT.


## Verification

