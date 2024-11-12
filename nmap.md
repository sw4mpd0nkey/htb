
# Nmap

## Nmap Architecture

Nmap offers many different types of scans that can be used to obtain various results about our targets. Basically, Nmap can be divided into the following scanning techniques:

- Host discovery
- Port scanning
- Service enumeration and detection
- OS detection
- Scriptable interaction with the target service (Nmap Scripting Engine)


## Scan Techniques

 - -sS/sT/sA/sW/sM: TCP SYN/Connect()/ACK/Window/Maimon scans
 - -sU: UDP Scan
 -sN/sF/sX: TCP Null, FIN, and Xmas scans
 - --scanflags <flags>: Customize TCP scan flags
 - -sI <zombie host[:probeport]>: Idle scan
 - -sY/sZ: SCTP INIT/COOKIE-ECHO scans
 - -sO: IP protocol scan
 - -b <FTP relay host>: FTP bounce scan

## Using Nmap

 ### Host discovery
 FIRST WE WANT TO KNOW IF HOST IS UP, NORMALLY IN A SUBDOWMAIN WE SCAN

 > sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5

 ### Port Scanning
 After we have found out that our target is alive, we want to get a more accurate picture of the system. The information we need includes:

  - Open ports and its services
  - Service versions
  - Information that the services provided
  - Operating system
 There are a total of 6 different states for a scanned port we can obtain:

 |State|Description|
 |-----|-----------|
 |open|This indicates that the connection to the scanned port has been established. These connections can be TCP connections, UDP datagrams as well as SCTP associations.|
 |closed|When the port is shown as closed, the TCP protocol indicates that the packet we received back contains an RST flag. This scanning method can also be used to determine if our target is alive or not.|
 |filtered|Nmap cannot correctly identify whether the scanned port is open or closed because either no response is returned from the target for the port or we get an error code from the target.|
 |unfiltered|This state of a port only occurs during the TCP-ACK scan and means that the port is accessible, but it cannot be determined whether it is open or closed.|
 |open/filtered|If we do not get a response for a specific port, Nmap will set it to that state. This indicates that a firewall or packet filter may protect the port.|
 |closed/filtered|This state only occurs in the IP ID idle scans and indicates that it was impossible to determine if the scanned port is closed or filtered by a firewall.|

## Nmap Scripting Engine
|Category|Description|
|--------|-----------|
|auth|Determination of authentication credentials.|
|broadcast|Scripts, which are used for host discovery by broadcasting and the discovered hosts, can be automatically added to the remaining scans.|
|brute|Executes scripts that try to log in to the respective service by brute-forcing with credentials.|
|default|Default scripts executed by using the -sC option.|
|discovery|Evaluation of accessible services.|
|dos|These scripts are used to check services for denial of service vulnerabilities and are used less as it harms the services.|
|exploit|This category of scripts tries to exploit known vulnerabilities for the scanned port.|
|external|Scripts that use external services for further processing.|
|fuzzer|This uses scripts to identify vulnerabilities and unexpected packet handling by sending different fields, which can take much time.|
|intrusive|Intrusive scripts that could negatively affect the target system.|
|malware|Checks if some malware infects the target system.|
|safe|Defensive scripts that do not perform intrusive and destructive access.|
|version|Extension for service detection.|
|vuln|Identification of specific vulnerabilities.|

## Firewalls and IDS/IPS
 Nmap's TCP ACK scan (-sA) method is much harder to filter for firewalls and IDS/IPS systems than regular SYN (-sS) or Connect scans (sT) because they only send a TCP packet with only the ACK flag. When a port is closed or open, the host must respond with an RST flag. Unlike outgoing connections, all connection attempts (with the SYN flag) from external networks are usually blocked by firewalls. However, the packets with the ACK flag are often passed by the firewall because the firewall cannot determine whether the connection was first established from the external network or the internal network.

 Example Ack Scan:
 > sudo nmap 10.129.2.28 -p 21,22,25 -sA -Pn -n --disable-arp-ping --packet-trace

 ### Decoys
 There are cases in which administrators block specific subnets from different regions in principle. This prevents any access to the target network. Another example is when IPS should block us. For this reason, the Decoy scanning method (-D) is the right choice. With this method, Nmap generates various random IP addresses inserted into the IP header to disguise the origin of the packet sent. With this method, we can generate random (RND) a specific number (for example: 5) of IP addresses separated by a colon (:). Our real IP address is then randomly placed between the generated IP addresses. 

 > sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5

## DNS Proxying
 Nmap still gives us a way to specify DNS servers ourselves (--dns-server <ns>,<ns>). This method could be fundamental to us if we are in a demilitarized zone (DMZ). The company's DNS servers are usually more trusted than those from the Internet. So, for example, we could use them to interact with the hosts of the internal network. As another example, we can use TCP port 53 as a source port (--source-port) for our scans. If the administrator uses the firewall to control this port and does not filter IDS/IPS properly, our TCP packets will be trusted and passed through.
 > sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53

// finding some sneaky dns stuff
sudo nmap -sSU -p 53 --script dns-nsid 10.129.113.182 -Pn -n --disable-arp-ping --packet-trace -D RND:50 --max-retries 50 --version-intensity 9 -sV


sudo nmap 10.129.31.116 -sSU -sV  -Pn -n --disable-arp-ping

//hard
 nmap -v -sV -p- -Pn -n --disable-arp-ping --source-port 53 <IP> 
  nmap <IP> -p50000 -sS -Pn -n --disable-arp-ping --packet-trace

nmap <IP> -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53 

nc -nv -p53 <IP> <target_port>


 10.129.135.77 




//hard - almost
 sudo nmap 10.129.135.77 -sA -sV -Pn -n --disable-arp-ping --source-port 22 -p50000 
//hard answer 
 sudo nmap 10.129.135.77 -p50000 -sS -sV -Pn -n --disable-arp-ping --packet-trace --source-port 53



ncat -nv --source-port 22 10.129.2.28 50000

HTB{kjnsdf2n982n1827eh76238s98di1w6}