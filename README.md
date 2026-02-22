# Troubeshooting-No-route-to-Host-Kali-on-proxmox
Resolved gateway reachability issues by identifying and correcting a missing VLAN xyz trunk on the switch, validated via ARP/tcpdump analysis
### Problem Summary

A Kali Linux VM configured on VLAN xyz could not reach its default gateway (10.161.X.X) or the internet.
Symptoms included:
- Destination Host Unreachable
- No route to host
- ARP table showing (incomplete) for the gateway
- No ICMP replies from the ASA
Other servers on VLAN X worked normally, which initially suggested the ASA and Proxmox were functioning correctly.


  
### 2. Initial Observations

Kali network configuration
- IP: 10.161.X.X/Z
- Gateway: 10.X.X.X
- Interface: eth0 (UP)
- Routing table: correct

Key finding
arp -n showed: 10.161.X.X (incomplete) > This indicated a Layer 2 failure, not routing or firewall


### 3. Deep Dive: Layer 2 Verification

Using sudo tcpdump -ni eth0 arp , kali was sending ARP request  but no ARP replies, confirming the ASA never received Kali's Traffic

### 4. Infrastructure Review

Proxmox Bridges
• 	 → connected to nic1mngt (physical trunk to switch)
• 	 → connected to nic0
Servers on VLAN X were on Vnet and worked fine.

Critical discovery was configured with VLAN IDs:9X , 
This meant:
• 	VLAN 9X was allowed
• 	VLAN xyz was blocked
• 	Kali’s tagged traffic never left the bridge
• 	ASA never saw ARP from Kali

### 5. Resolution

Updated the physical switch port to allow VLAN xyz on the trunk and updated Proxmox Vnet to include VLAN xyz.
After applying the fix  > - ASA ARP table showed Vlan xyz 10.161.X.X  bc24.11af.--- and Kali tcpdump showed ARP replies


### Lessons Learned
• 	ARP incomplete is a strong indicator of VLAN or Layer 2 isolation.
• 	Proxmox bridges must explicitly allow all VLANs used by VMs.
• 	Physical switch trunks must match Proxmox VLAN configuration.
• 	ASA subinterfaces only work if the VLAN reaches the ASA at Layer 2.
• 	Always validate the full path: VM → Bridge → NIC → Switch → ASA.

### Skills demonstrated
🔹 	Network Troubleshooting & Diagnostics
- Diagnosed gateway reachability issues by analyzing ARP behavior, routing tables, and interface states.
- Distinguished between L2 failures (ARP incomplete) and L3 symptoms (ICMP unreachable).

🔹	VLAN & Trunking Configuration
- Validated and corrected VLAN tagging across:
- Proxmox VM NICs
- Linux bridges
- Physical switch trunk ports
- ASA subinterfaces
- Identified a missing VLAN xyz trunk as the root cause of ARP failure

🔹 Firewall & ASA Subinterface Management
- Verified ASA subinterface configuration for multiple VLANs.
- Confirmed ASA ARP responses and interface status to validate L2 connectivity.
- Ensured ASA routing and NAT readiness after restoring VLAN reachability

🔹 	Packet Capture & Traffic Analysis
- Used tcpdump to observe ARP requests and replies in real time.
- Interpreted packet-level evidence to confirm VLAN path restoration.
- Demonstrated SOC-relevant traffic inspection and diagnostic skills


