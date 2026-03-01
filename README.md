# Virtualized Network Security Lab: PfSense Firewall + OpenVPN

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
  3. ENABLING MY HOST TO ACCESS THE PFSENSE WEB UI. To ensure that my host could access the pfSense Web UI (which would be needed to know my VPN could work), I needed to do two things. First, I needed to change the LAN adapter for my pfSense VM and all other VMs to "Host-only" so that the host (my PC for this lab, in other cases, this could be a remote device/workstation) could access my VMs (IM5). Second, I had to enable a host adapter. I did this my going into VirtualBox, clicking on tools in the top left corner, properties, then choose one of the avalabile host only network to configure. This would be the adapter my host would use to connect to my web UI and VPN. I chose the ip address of 10.10.10.254, as it was on the same subnet as my other VMs ip's (IM6). After configuring this, I was able to access the pfSense web UI from my host PC (IM7).
  4. INSTALLING OPENVPN CLIENT EXPORT PACKAGE. One of the advantages of having pfSense is that the openVPN server is already built in, meaning all that I had to do is install the export package within pfSense to obtain all the configurations I needed to set up my VPN. All I had to do was go into the pfSense UI and go to the packages section to see all the available packages. I went to the search bar and typed "openvpn," and the specific package that I needed was shown. I installed it, and after about 30 seconds, I had all the files I needed to start creating my VPN server (IM8).

<img width="562" height="50" alt="Screenshot 2026-02-24 171406" src="https://github.com/user-attachments/assets/ccb468cf-f25a-4ee0-9f33-f4b51a5bb36e" />

IM1: pfSense VM configured with WAN and LAN interfaces

<img width="516" height="147" alt="Screenshot 2026-02-24 171457" src="https://github.com/user-attachments/assets/dd6ea5df-48ee-4c9a-b7dc-bd26b8995695" />

IM2: Pinging Windows from Kali Linux

<img width="502" height="111" alt="Screenshot 2026-02-24 171535" src="https://github.com/user-attachments/assets/21e63f48-aee8-4ab2-b651-81944cbe9db4" />

IM3: Pinging Kali Linux from Ubuntu

<img width="448" height="278" alt="Screenshot 2026-02-24 171623" src="https://github.com/user-attachments/assets/665e3bde-a3b4-43d0-9bd5-177290952264" />

IM4: Pinging Ubuntu from Windows

<img width="551" height="265" alt="Screenshot 2026-02-24 171803" src="https://github.com/user-attachments/assets/1a66f0a3-d2b2-47a5-88d4-c372772e07bf" />

IM5: Enabling Host-Only adapter option in VirtualBox

<img width="1395" height="899" alt="image" src="https://github.com/user-attachments/assets/9fc30e24-76bf-4fe4-b241-3a2538007367" />

IM6: Configuring the host adapter ip address and subnet mask

<img width="1536" height="813" alt="Screenshot 2026-02-24 172053" src="https://github.com/user-attachments/assets/54d71397-2bb9-4752-9112-59369a8b661b" />

IM7: Accessing pfSense web UI from my host PC

<img width="1138" height="100" alt="Screenshot 2026-02-24 172226" src="https://github.com/user-attachments/assets/9a185ec6-299d-4522-87d9-ac32ddeb6fd7" />

IM8: Downloaded OpenVPN export Configuration Files

### Step 1: Configuring OpenVPN Server
  Now that all areas of my Lab environment were correctly configured, I was ready to start setting up my VPN. The first thing I had to do was to create and configure the OpenVPN server. Here are the steps I took to complete this.
  1. CREATING CERTIFICATE AUTHORITY AND SERVER CERTIFICATE: This is the first step in setting up my OpenVPN server. A certificate authority for this scenario is essentially my root of trust. It digitally signs off on anything that goes through my VPN so that it is trusted. I needed to create a Certificate Authority because OpenVPN uses TLS (Transport Layer Security), which requires that both the OpenVPN server and the client (me in this case) are correctly identified. Without a CA, any client could try and access my workstations or pretend to be my VPN server. The reason for creating the server certificate is to ensure that the VPN server is actually who it says it is. This prevents fake VPN servers from trying to gain remote access to my system or any system, for that matter. I named the CA "internal CA." I used AES-256 as my encryption algorithm for this server. The configurations for the server can be seen in IM9.
  2. CHOOSING PROTOCOL AND PORT#: Next, I had to configure the protocol and port number for my OpenVPN server. For the protocol, I chose UDP (User Datagram Protocol). I chose UDP, it has lower overhead compared to TCP, and it is faster for real-time ecrypted traffic. TCP is still an option for an OpenVPN server, but I chose UDP. I chose port 1194 for OpenVPN. The reason I chose this is that port 1194 is the default port for OpenVPN servers. This was nice and simple and didn't need much thought. These configurations can be seen in IM10.
  3. CONFIGURING TUNNEL SETTINGS: Tunnel settings for this server needed to be configured because the VPN needs its own "virtual network." This is because if the VPN server is routed right to my LAN network of 10.10.10.0/24, it would create ip   conflicts, and pfSense wouldn't be able to distinguish between LAN and VPN network traffic. Because of this, I created a tunnel network for the VPN so it could route separately. The configuration is noted in IM11.
  4. CREATE VPN USER: Next, I had to create a user for the VPN. The reason for this is that without a VPN user identity, the server would have no idea who was trying to log in and gain remote access. This server was configured with user + certificate authentication, so a VPN user was necessary. For simplicity, named the user "vpnuser" and assigned a password (IM12).
  5. EXPORT CLIENT CONFIGURATION FILE: The last thing in setting up my VPN was exporting the client configuration file. By doing this, I could access the OpenVPN application and test to see if I could get remote access. To do this, I went to the server settings and scrolled down to the clients. There I saw the vpnuser I created, and it showed to its right the specific install options depending on my OS (IM13). After this, I went through the installer wizard to complete the installation, and I had my VPN ready to go.

<img width="913" height="773" alt="Screenshot 2026-02-24 184803" src="https://github.com/user-attachments/assets/deceb4aa-10db-462d-9c69-2e01e56e07b9" />

IM9: Certificate Authority and Server Certificate configuration

<img width="1906" height="807" alt="Screenshot 2026-02-24 184703" src="https://github.com/user-attachments/assets/a89824b4-48b2-4d03-8661-8453d3a1f59a" />

IM10: Protocol and Port Configuration

<img width="908" height="300" alt="Screenshot 2026-02-24 184824" src="https://github.com/user-attachments/assets/eb0efc40-d7f9-4949-8e17-39a914aa53ce" />

IM11: Tunnel Settings Configuration

<img width="1141" height="236" alt="Screenshot 2026-02-24 190701" src="https://github.com/user-attachments/assets/6a206a43-375c-4ed2-b4ae-744d37e08867" />

IM12: Creating VPN user

<img width="1137" height="286" alt="Screenshot 2026-02-24 190722" src="https://github.com/user-attachments/assets/4688fa6c-20ba-4101-93c1-4e3e55dceab7" />

IM13: Exporting Client Configuration File

### Step 2: Testing VPN Connectivity: 
  Once my OpenVPN server was configured and I downloaded all the necessary files, my VPN was ready to go. From this point on, you could really try anything with the VPN. Everything up until this point was necessary configurations. Now I wanted to test specific things to see if my VPN was actually working. 
  1. CONNECT VPN CLIENT: The first and most important check was to see if my VPN was actually able to connect. I tested this by simply loading up the VPN. I opened the OpenVPN GUI application, then a small monitor icon opened in the taskbar of my host. I right-clicked, chose the "connect" option, then the OpenVPN connection interface opened. When correctly connected, it should read "Initialization Sequence Completed," and that is exactly what I saw (IM14). My VPN succesfully Connected to my internal lab and VMs.
  2. PINGING A VM TO CONFIRM INTERNAL CONNECTIVITY: To ensure that my host could connect and communicate with my VMs, I tested to see if my host could ping my pfSense LAN ip address (10.10.10.1). i Went into the terminal as an administrator and pinged the pfSense LAN ip, and the ping was successful, ensuring that my host was connected to my internal VMs (IM15).
  3. USING SSH TO REMOTELY ACCESS KALI LINUX: The last thing I did to test connectivity to my VMs was to use SSH to remotely access my Kali Linux VM from my host machine. First, I had to check if SSH was enabled on my Kali Linux VM. I did this by going into the terminal and typing in a few commands. First, I had to check the status of ssh on my VM by typing "sudo systemctl status ssh." The result showed that SSH was disabled. For your lab setup or network, it is up to you whether you want to enable specific ports/protocols. Leaving specific ports open when unnecessary could result in vulnerabilities being exploited. For this lab, however, I needed to enable ssh so I could get remote access from my host. I then typed "sudo systemctl start ssh" to start the application, then ran "sudo systemctl enable ssh" to enable ssh. Then I checked the status again, and it was up and running, signaled by the "active (running)" text in green (IM16). Once that was done, I went to a terminal on my host machine and used the command "ssh kali@10.10.10.101" to try to gain remote access to Kali Linux from my host. "ssh" is used to specifiy the service/application to use, "kali" is the username for my Kali Linux VM, and "@10.10.10.101" is used to specifiy the target to ssh into, in this case the ip address of my Kali Linux VM. After entering the command, I was asked to type in the password for the Kali Linux account, and then I was in. The result showed a command prompt exactly like the one if I were in Kali Linux itself, proving that I successfully gained remote access to my Kali Linux VM from my host (IM17).

<img width="1072" height="830" alt="Screenshot 2026-02-26 170739" src="https://github.com/user-attachments/assets/10d0a20c-6921-44e8-afb6-03acc54437cd" />

IM14: Successful connection for VPN

<img width="516" height="170" alt="Screenshot 2026-02-24 171924" src="https://github.com/user-attachments/assets/dc6c7ead-a594-4a39-89cc-9df344b95a96" />

IM15: Successful ping from host machine to pfSense VM 

<img width="846" height="388" alt="Screenshot 2026-02-26 184606" src="https://github.com/user-attachments/assets/46132450-cf3e-402a-884c-38de08a3adb9" />

IM16: Commands used to start and enable SSH on Kali Linux

<img width="500" height="100" alt="Screenshot 2026-02-26 185133" src="https://github.com/user-attachments/assets/9784d8a1-d29a-41fd-9492-c9f3fa4e90c5" />

IM17: Successful SSH connection from the host machine to Kali Linux

### Step 3: Extra practices
  Now That everthing was set up and working correctly, I wanted to try other things to practice with my VPN. 
    1. CAPTURING VPN TRAFFIC IN WIRESHARK: Wireshark is an open source and free protocol analyzer and packet-capturing tool used to capture and interactively browse the network traffic on computer networks in real time. It enables IT professionals to troubleshoot, analyze, and debug network problems by examining information from hundreds of protocols in minute detail. I wanted to use it to inspect the network traffic coming from my VPN. I opened the wireshark application on my host, clicked on the correct adapter the VPN was using, and started the packet capture. I had to put in a specific filter to easily find the traffic I was looking for, which was udp for this case, as that is how I configured my OpenVPN server. After about 30 seconds, I stopped the capture and looked at the results. I saw 10 packets (IM18). It can be seen that the two hosts that are communication is my pfSense LAN interface (10.10.10.1), and the IP address I configured for my host-only adapter on my host machine (10.10.10.254). We can also see the protocol is OpenVPN, so it is the correct capture. I then took a look at a specific packet to investigate its qualities (IM19). Under the IPv4 header, we can see the source and destination ip's as mentioned. Under the UDP header, we can see the source port is the same as the one I assigned to the OpenVPN server. 

   <img width="700" height="542" alt="Screenshot 2026-02-26 194051" src="https://github.com/user-attachments/assets/7c79e74b-1d43-40cd-9fd2-f0a6d5bc5e7c" />

   IM18: Wireshark Capture of OpenVPN server communication

   <img width="938" height="302" alt="Screenshot 2026-02-26 194133" src="https://github.com/user-attachments/assets/ab330b83-59bd-42e3-ae95-a4d1a7101e8c" />

   IM19: IP and UDP Information under a specific packet in Wireshark


   ## Conclusion: 
   This Virtual Network Security Lab was successful in its attempt to simulate a real-world enterprise edge network environment using the tools pfSense and OpenVPN. By employing various techniques such as network segmentation, interface-based firewall rules, and certificate-based VPN authentication, I was able to create a secure routed network infrastructure, which is similar to a real-world network security environment. 

  Through the successful completion of this project, I have further enhanced my knowledge of routing logic, the processing of firewall rules, the implementation of PKIs, and the configuration of secure remote access infrastructure. Troubleshooting VPN connectivity, TLS handshake, and subnet communication issues has further enhanced my ability to troubleshoot real-world networking issues in a systematic manner.

  This project has been successful in its attempt to bridge the gap between the theoretical concepts of network security and its practical implementation, which can be used to secure enterprise networks as well as remote access infrastructure.


   

   




    















































