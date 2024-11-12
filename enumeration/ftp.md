# File Transfer Protocol

The File Transfer Protocol (FTP) is one of the oldest protocols on the Internet. The FTP runs within the application layer of the TCP/IP protocol stack. Thus, it is on the same layer as HTTP or POP. These protocols also work with the support of browsers or email clients to perform their services. There are also special FTP programs for the File Transfer Protocol. 

In an FTP connection, two channels are opened. First, the client and server establish a control channel through TCP port 21. The client sends commands to the server, and the server returns status codes. Then both communication participants can establish the data channel via TCP port 20. This channel is used exclusively for data transmission, and the protocol watches for errors during this process. If a connection is broken off during transmission, the transport can be resumed after re-established contact.

## TFTP

Trivial File Transfer Protocol (TFTP) is simpler than FTP and performs file transfers between client and server processes. However, it does not provide user authentication and other valuable features supported by FTP. In addition, while FTP uses TCP, TFTP uses UDP, making it an unreliable protocol and causing it to use UDP-assisted application layer recovery.

Let us take a look at a few commands of TFTP:

|Commands|Description|
|--------|-----------|
|connect|Sets the remote host, and optionally the port, for file transfers.|
|get|Transfers a file or set of files from the remote host to the local host.|
|put|Transfers a file or set of files from the local host onto the remote host.|
|quit|Exits tftp.|
|status|Shows the current status of tftp, including the current transfer mode (ascii or binary), connection status, time-out value, and so on.|
|verbose|Turns verbose mode, which displays additional information during file transfer, on or off.|

### Default Configuration

One of the most used FTP servers on Linux-based distributions is vsFTPd. The default configuration of vsFTPd can be found in /etc/vsftpd.conf, and some settings are already predefined by default. It is highly recommended to install the vsFTPd server on a VM and have a closer look at this configuration.

Viewing vsFTPd Config File

> cat /etc/vsftpd.conf | grep -v "#"

will show something like the following:
|Setting|Description|
|-------|------------|
|listen=NO|Run from inetd or as a standalone daemon?|
|listen_ipv6=YES|Listen on IPv6 ?|
|anonymous_enable=NO|Enable Anonymous access?|
|local_enable=YES|Allow local users to login?|
|dirmessage_enable=YES|Display active directory messages when users go into certain directories?|
|use_localtime=YES|Use local time?|
|xferlog_enable=YES|Activate logging of uploads/downloads?|
|connect_from_port_20=YES|Connect from port 20?|
|secure_chroot_dir=/var/run/vsftpd/empty|Name of an empty directory|
|pam_service_name=vsftpd|This string is the name of the PAM service vsftpd will use.
|rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem|The last three options specify |the location of the RSA certificate to use for SSL encrypted connections.|
|rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key|...| 	
|ssl_enable=NO|...|

### FTPUSERS
In addition, there is a file called /etc/ftpusers that we also need to pay attention to, as this file is used to deny certain users access to the FTP service. In the following example, the users guest, john, and kevin are not permitted to log in to the FTP service, even if they exist on the Linux system.

```
frank-106@htb[/htb]$ cat /etc/ftpusers

guest
john
kevin
```

## Dangerous Settings

There are many different security-related settings we can make on each FTP server. These can have various purposes, such as testing connections through the firewalls, testing routes, and authentication mechanisms. One of these authentication mechanisms is the anonymous user. This is often used to allow everyone on the internal network to share files and data without accessing each other's computers. With vsFTPd, the optional settings that can be added to the configuration file for the anonymous login look like this:

|Setting|Description|
|-------|-----------|
|anonymous_enable=YES|Allowing anonymous login?|
|anon_upload_enable=YES|Allowing anonymous to upload files?|
|anon_mkdir_write_enable=YES|Allowing anonymous to create new directories? |
|no_anon_password=YES|Do not ask anonymous for password?|
|anon_root=/home/username/ftp|Directory for anonymous.|
|write_enable=YES|Allow the usage of FTP commands: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, and SITE?|

## Anonymous Login

Some FTP endpoints are set up to allow anonymous login:
```
frank-106@htb[/htb]$ ftp 10.129.14.136

Connected to 10.129.14.136.
220 "Welcome to the HTB Academy vsFTP service."
Name (10.129.14.136:cry0l1t3): anonymous

230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.


ftp> ls

200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Clients
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
226 Directory send OK.
```
Note: might need to enter anonymous:anonymous 

## Footprinting the Service

Footprinting using various network scanners is also a handy and widespread approach. These tools make it easier for us to identify different services, even if they are not accessible on standard ports. One of the most widely used tools for this purpose is Nmap. Nmap also brings the Nmap Scripting Engine (NSE), a set of many different scripts written for specific services. More information on the capabilities of Nmap and NSE can be found in the Network Enumeration with Nmap module. We can update this database of NSE scripts with the command shown.

### Nmap FTP Scripts

Updating nmap script db:

```
frank-106@htb[/htb]$ sudo nmap --script-updatedb

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 13:49 CEST
NSE: Updating rule database.
NSE: Script Database updated successfully.
Nmap done: 0 IP addresses (0 hosts up) scanned in 0.28 seconds
```

All the NSE scripts are located on the Pwnbox in /usr/share/nmap/scripts/, but on our systems, we can find them using a simple command on our system.

```
frank-106@htb[/htb]$ find / -type f -name ftp* 2>/dev/null | grep scripts

/usr/share/nmap/scripts/ftp-syst.nse
/usr/share/nmap/scripts/ftp-vsftpd-backdoor.nse
/usr/share/nmap/scripts/ftp-vuln-cve2010-4221.nse
/usr/share/nmap/scripts/ftp-proftpd-backdoor.nse
/usr/share/nmap/scripts/ftp-bounce.nse
/usr/share/nmap/scripts/ftp-libopie.nse
/usr/share/nmap/scripts/ftp-anon.nse
/usr/share/nmap/scripts/ftp-brute.nse
```

### Nmap

Default script scan
```
frank-106@htb[/htb]$ sudo nmap -sV -p21 -sC -A 10.129.14.136

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-16 18:12 CEST
Nmap scan report for 10.129.14.136
Host is up (0.00013s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rwxrwxrwx    1 ftp      ftp       8138592 Sep 16 17:24 Calendar.pptx [NSE: writeable]
| drwxrwxrwx    4 ftp      ftp          4096 Sep 16 17:57 Clients [NSE: writeable]
| drwxrwxrwx    2 ftp      ftp          4096 Sep 16 18:05 Documents [NSE: writeable]
| drwxrwxrwx    2 ftp      ftp          4096 Sep 16 17:24 Employees [NSE: writeable]
| -rwxrwxrwx    1 ftp      ftp            41 Sep 16 17:24 Important Notes.txt [NSE: writeable]
|_-rwxrwxrwx    1 ftp      ftp             0 Sep 15 14:57 testupload.txt [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.4
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
```

### Nmap Script Trace

The scan history shows that four different parallel scans are running against the service, with various timeouts. For the NSE scripts, we see that our local machine uses other output ports (54226, 54228, 54230, 54232) and first initiates the connection with the CONNECT command. From the first response from the server, we can see that we are receiving the banner from the server to our second NSE script (54228) from the target FTP server. If necessary, we can, of course, use other applications such as netcat or telnet to interact with the FTP server.

```
frank-106@htb[/htb]$ sudo nmap -sV -p21 -sC -A 10.129.14.136 --script-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 13:54 CEST                                                                                                                                                   
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 8 [10.129.14.136:21]                                   
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 16 [10.129.14.136:21]             
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 24 [10.129.14.136:21]
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 32 [10.129.14.136:21]
NSOCK INFO [11.4640s] nsock_read(): Read request from IOD #1 [10.129.14.136:21] (timeout: 7000ms) EID 42
NSOCK INFO [11.4640s] nsock_read(): Read request from IOD #2 [10.129.14.136:21] (timeout: 9000ms) EID 50
NSOCK INFO [11.4640s] nsock_read(): Read request from IOD #3 [10.129.14.136:21] (timeout: 7000ms) EID 58
NSOCK INFO [11.4640s] nsock_read(): Read request from IOD #4 [10.129.14.136:21] (timeout: 11000ms) EID 66
NSE: TCP 10.10.14.4:54226 > 10.129.14.136:21 | CONNECT
NSE: TCP 10.10.14.4:54228 > 10.129.14.136:21 | CONNECT
NSE: TCP 10.10.14.4:54230 > 10.129.14.136:21 | CONNECT
NSE: TCP 10.10.14.4:54232 > 10.129.14.136:21 | CONNECT
NSOCK INFO [11.4660s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 50 [10.129.14.136:21] (41 bytes): 220 Welcome to HTB-Academy FTP service...
NSOCK INFO [11.4660s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 58 [10.129.14.136:21] (41 bytes): 220 Welcome to HTB-Academy FTP service...
NSE: TCP 10.10.14.4:54228 < 10.129.14.136:21 | 220 Welcome to HTB-Academy FTP service.
```

### Service Interaction

Normally you can connect with ftp client, can also try netcat or telnet to connect, grab banners, etc

netcat
>  nc -nv 10.129.14.136 21

telnet
> telnet 10.129.14.136 21

It looks slightly different if the FTP server runs with TLS/SSL encryption. Because then we need a client that can handle TLS/SSL. For this, we can use the client openssl and communicate with the FTP server. The good thing about using openssl is that we can see the SSL certificate, which can also be helpful.

> openssl s_client -connect 10.129.14.136:21 -starttls ftp

