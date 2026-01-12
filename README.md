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
1. Create your Customer LANs
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
Each customer router uses a default route pointing to its directly connected CGNAT gateway interface
<ul>
<li>Customer 1 Router <code>ip route 0.0.0.0 0.0.0.0 100.64.10.2</code></li>
<li>Customer 2 Router <code>ip route 0.0.0.0 0.0.0.0 100.64.10.6</code></li>
</ul>

### CGNAT Gateway Configuration
The CGNAT gateway handles the inside and outside NAT configuration. The inside NAT will translate subscriber data into 
