# Virtualized-Network-Security-Lab-pfSense-Firewall-OpenVPN

## Objective

The goal of the Virtual Enterprise Network Security Lab project was to create a fully segmented virtual network environment utilizing a pfSense firewall and an OpenVPN remote access solution. The key objectives were to set up internal network segmentation, firewall rules, NAT, and certificate-based VPN authentication to mimic a real-world enterprise edge network scenario. This hands-on lab experience has provided me with a better understanding of routing, firewall rule processing, public key infrastructure (PKI), and secure remote access while learning to solve real-world networking issues.

### Skills Learned


- Advanced knowledge of firewall configuration, interface-based rule sets, and network segmentation utilizing pfSense.
- Practical experience in implementing and troubleshooting a certificate-based OpenVPN solution for remote access VPN.
- Familiarity with configuring routing, NAT, and communication between multiple networks within a virtualized environment.
- Development of enhanced knowledge of VPN tunneling, TLS authentication, and encrypted remote access.
- Development of structured troubleshooting skills through resolving TLS handshake failures, routing problems, and firewall configuration issues.

### Tools Used


-VirtualBox -pfSense -OpenVPN -VirtualBox VMs (Windows 10 server, Kali Linux, Linux Ubuntu) OpenVPN GUI (for VPN tunnels from host) -Command-Line tools (ping,ipconfig,nslookup, etc)

## Steps/Procedure

### Step 0: Pre-lab Checklist/Setup
Before Starting This VPN lab, I had to ensure that my lab environment was set up correctly to ensure that everything would operate smoothly and the VPN would connect with pfSense. There was a few things I had to set up. 

  1. PFSENSE WAS CORRECTLY CONFIGURED WITH A WAN AND LAN INTERFACE (IM1). I already had pfSense configured from a previous lab, with a WAN interface configured with DHCP (Dynamic Host Configuration Protocol) so my VMs could reach the internet and a LAN interface so my VMs could communicate and interact with eachother.
  2. ENSURING MY INTERNAL LAB NETWORK WAS FUNCTIONING CORRECTLY. The main way I tested this was by using the ping command (both in Linux and Windows) to test connectivity between VMs (In the command prompt for Windows and in the terminal for Linux). First, I pinged my Windows VM from Kali Linux (IM2), then I pinged Kali Linux from Ubuntu (IM3), and lastly, I pinged my Ubuntu VM from Windows (IM4). All the pings were successful, ensuring that all my VMs could communicate with eachother and were in the same internal network.
  3. ENABLING MY HOST TO ACCESS THE PFSENSE WEB UI. To ensure that my host could access the pfSense Web UI (which would be needed to know my VPN could work), I needed to do two things. First, I needed to change the LAN adapter for my pfSense VM and all other VMs to "Host-only" so that the host (my PC for this lab, in other cases, this could be a remote device/workstation) could access my VMs (IM5). Second, I had to enable a host adapter. I did this my going into VirtualBox, clicking on tools in the top left corner, properties, then choose one of the avalabile host only network to configure. This would be the adapter my host would use to connect to my web UI and VPN. I chose the ip address of 10.10.10.254, as it was on the same subnet as my other VMs ip's (IM6). 

