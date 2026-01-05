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
 <li><b>NOTE: </b>LAN Routers have been configured as DHCP server to provide endpoints in the LANs with IP addresses. To see how to configure DHCP on a router (Router-on-a-stick configuration), see this <a href="https://nodeconnect.blogspot.com/2025/04/how-to-set-up-cisco-router-as-dhcp.html">post</a> </li>
</ul>
  
<ul>
  <li>Choose a subnet for your LANs</li>
  <li>Give each device an IP address and establish your default gateway</li>
  **NOTE** LAN Routers used DHCP. You can also statically configure your IPs for PCs. Ensure it is in the same subnet as your configured default gateway. 
  <li>Verify IP connectivity</li>
</ul>
<b>Troubleshooting</b>
<ul>
  <li></li>
</ul>
