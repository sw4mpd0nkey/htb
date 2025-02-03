# Introduction to Windows Privilege Escalation

---

After gaining a foothold, elevating our privileges will provide more options for persistence and may reveal information stored locally that can further our access within the environment. The general goal of Windows privilege escalation is to further our access to a given system to a member of the `Local Administrators` group or the `NT AUTHORITY\SYSTEM` [LocalSystem](https://docs.microsoft.com/en-us/windows/win32/services/localsystem-account) account. There may, however, be scenarios where escalating to another user on the system may be enough to reach our goal. Privilege escalation is typically a vital step during any engagement. We need to use the access obtained, or some data (such as credentials) found only once we have a session in an elevated context. In some cases, privilege escalation may be the ultimate goal of the assessment if our client hires us for a "gold image" or "workstation breakout" type assessment. Privilege escalation is often vital to continue through a network towards our ultimate objective, as well as for lateral movement.

That being said, we may need to escalate privileges for one of the following reasons:

|  |  |
| --- | --- |
| 1. | When testing a client's [gold image](https://www.techopedia.com/definition/29456/golden-image) Windows workstation and server build for flaws |
| 2. | To escalate privileges locally to gain access to some local resource such as a database |
| 3. | To gain [NT AUTHORITY\\System](https://docs.microsoft.com/en-us/windows/win32/services/localsystem-account) level access on a domain-joined machine to gain a foothold into the client's Active Directory environment |
| 4. | To obtain credentials to move laterally or escalate privileges within the client's network |

There are many tools available to us as penetration testers to assist with privilege escalation. Still, it is also essential to understand how to perform privilege escalation checks and leverage flaws `manually` to the extent possible in a given scenario. We may run into situations where a client places us on a managed workstation with no internet access, heavily firewalled, and USB ports disabled, so we cannot load any tools/helper scripts. In this instance, it would be crucial to have a firm grasp of Windows privilege escalation checks using both PowerShell and Windows command-line.

Windows systems present a vast attack surface. Just some of the ways that we can escalate privileges are:

|  |  |
| --- | --- |
| Abusing Windows group privileges | Abusing Windows user privileges |
| Bypassing User Account Control | Abusing weak service/file permissions |
| Leveraging unpatched kernel exploits | Credential theft |
| Traffic Capture | and more. |

---

## Scenario 1 - Overcoming Network Restrictions

I once was given the task to escalate privileges on a client-provided system with no internet access and blocked USB ports. Due to network access control in place, I could not plug my attack machine directly into the user network to assist me. During the assessment, I had already found a network flaw in which the printer VLAN was configured to allow outbound communication over ports 80, 443, and 445. I used manual enumeration methods to find a permissions-related flaw that allowed me to escalate privileges and perform a manual memory dump of the `LSASS` process. From here, I was able to mount an SMB share hosted on my attack machine on the printer VLAN and exfil the `LSASS` DMP file. With this file in hand, I used `Mimikatz` offline to retrieve the NTLM password hash for a domain admin, which I could crack offline and use to access a domain controller from the client-provided system.

---

## Scenario 2 - Pillaging Open Shares

During another assessment, I found myself in a pretty locked-down environment that was well monitored and without any obvious configuration flaws or vulnerable services/applications in use. I found a wide-open file share, allowing all users to list its contents and download files stored on it. This share was hosting backups of virtual machines in the environment. I was explicitly interested in virtual harddrive files (`.VMDK` and `.VHDX` files). I could access this share from a Windows VM, mount the `.VHDX` virtual hard drive as a local drive and browse the file system. From here, I retrieved the `SYSTEM`, `SAM`, and `SECURITY` registry hives, moved them to my Linux attack box, and extracted the local administrator password hash using the [secretsdump.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py) tool. The organization happened to be using a gold image, and the local administrator hash could be used to gain admin access to nearly every Windows system via a pass-the-hash attack.

---

## Scenario 3 - Hunting Credentials and Abusing Account Privileges

In this final scenario, I was placed in a rather locked-down network with the goal of accessing critical database servers. The client provided me a laptop with a standard domain user account, and I could load tools onto it. I eventually ran the [Snaffler](https://github.com/SnaffCon/Snaffler) tool to hunt file shares for sensitive information. I came across some `.sql` files containing low-privileged database credentials to a database on one of their database servers. I used an MSSQL client locally to connect to the database using the database credentials, enable the [xp\_cmdshell](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver15) stored procedure and gain local command execution. Using this access as a service account, I confirmed that I had the [SeImpersonatePrivilege](https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege), which can be leveraged for local privilege escalation. I downloaded a custom compiled version of [Juicy Potato](https://github.com/ohpe/juicy-potato) to the host to assist with privilege escalation, and was able to add a local admin user. Adding a user was not ideal, but my attempts to obtain a beacon/reverse shell did not work. With this access, I was able to remote into the database host and gain complete control of one of the company's clients' databases.

---

## Why does Privilege Escalation Happen?

There is no one reason why a company's host(s) may fall victim to privilege escalation, but several possible underlying causes exist. Some typical reasons that flaws are introduced and go unnoticed are personnel and budget. Many organizations simply do not have the personnel to properly keep up with patching, vulnerability management, periodic internal assessments (self-assessments), continuous monitoring, and larger, more resource-intensive initiatives. Such initiatives may include workstation and server upgrades, as well as file share audits (to lock down directories and secure/remove sensitive files such as scripts or configuration files containing credentials).

---

## Moving On

The scenarios above show how an understanding of Windows privilege escalation is crucial for a penetration tester. In the real world, we will rarely be attacking a single host and need to be able to think on our feet. We should be able to find creative ways to escalate privileges and ways to use this access to further our progress towards the goal of the assessment.

---

## Practical Examples

Throughout the module, we will cover examples with accompanying command output, most of which can be reproduced on the target VMs that can be spawned within the relevant sections. You will be provided RDP credentials to interact with the target VMs and complete the section exercises and skills assessments. You can connect from the Pwnbox or your own VM (after downloading a VPN key once a machine spawns) via RDP using [FreeRDP](https://github.com/FreeRDP/FreeRDP/wiki/CommandLineInterface), [Remmina](https://remmina.org/), or the RDP client of your choice.

#### Connecting via FreeRDP

We can connect via command line using the command `xfreerdp /v:<target ip> /u:htb-student` and typing in the provided password when prompted. Most sections will provide credentials for the `htb-student` user, but some, depending on the material, will have you RDP with a different user, and alternate credentials will be provided.

Introduction to Windows Privilege Escalation

```shell-session
swmpdnky@htb[/htb]$  xfreerdp /v:10.129.43.36 /u:htb-student

[21:17:27:323] [28158:28159] [INFO][com.freerdp.core] - freerdp_connect:freerdp_set_last_error_ex resetting error state
[21:17:27:323] [28158:28159] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpdr
[21:17:27:324] [28158:28159] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpsnd
[21:17:27:324] [28158:28159] [INFO][com.freerdp.client.common.cmdline] - loading channelEx cliprdr
[21:17:27:648] [28158:28159] [INFO][com.freerdp.primitives] - primitives autodetect, using optimized
[21:17:27:672] [28158:28159] [INFO][com.freerdp.core] - freerdp_tcp_is_hostname_resolvable:freerdp_set_last_error_ex resetting error state
[21:17:27:672] [28158:28159] [INFO][com.freerdp.core] - freerdp_tcp_connect:freerdp_set_last_error_ex resetting error state
[21:17:28:770] [28158:28159] [INFO][com.freerdp.crypto] - creating directory /home/user2/.config/freerdp
[21:17:28:770] [28158:28159] [INFO][com.freerdp.crypto] - creating directory [/home/user2/.config/freerdp/certs]
[21:17:28:771] [28158:28159] [INFO][com.freerdp.crypto] - created directory [/home/user2/.config/freerdp/server]
[21:17:28:794] [28158:28159] [WARN][com.freerdp.crypto] - Certificate verification failure 'self signed certificate (18)' at stack position 0
[21:17:28:794] [28158:28159] [WARN][com.freerdp.crypto] - CN = WINLPE-SKILLS1-SRV
[21:17:28:795] [28158:28159] [ERROR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
[21:17:28:795] [28158:28159] [ERROR][com.freerdp.crypto] - @           WARNING: CERTIFICATE NAME MISMATCH!           @
[21:17:28:795] [28158:28159] [ERROR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
[21:17:28:795] [28158:28159] [ERROR][com.freerdp.crypto] - The hostname used for this connection (10.129.43.36:3389) 
[21:17:28:795] [28158:28159] [ERROR][com.freerdp.crypto] - does not match the name given in the certificate:
[21:17:28:795] [28158:28159] [ERROR][com.freerdp.crypto] - Common Name (CN):
[21:17:28:795] [28158:28159] [ERROR][com.freerdp.crypto] - 	WINLPE-SKILLS1-SRV
[21:17:28:795] [28158:28159] [ERROR][com.freerdp.crypto] - A valid certificate for the wrong name should NOT be trusted!
Certificate details for 10.129.43.36:3389 (RDP-Server):
	Common Name: WINLPE-SKILLS1-SRV
	Subject:     CN = WINLPE-SKILLS1-SRV
	Issuer:      CN = WINLPE-SKILLS1-SRV
	Thumbprint:  9f:f0:dd:28:f5:6f:83:db:5e:8c:5a:e9:5f:50:a4:50:2d:b3:e7:a7:af:f4:4a:8a:1a:08:f3:cb:46:c3:c3:e8
The above X.509 certificate could not be verified, possibly because you do not have
the CA certificate in your certificate store, or the certificate has expired.
Please look at the OpenSSL documentation on how to add a private CA to the store.
Do you trust the above certificate? (Y/T/N) y
Password: 
```

Many of the module sections require tools such as open-source scripts, precompiled binaries, and exploit PoCs. Where applicable, these can be found in the `C:\Tools` directory on the target host. Even though most tools are provided, challenge yourself to upload files to the target (using techniques showcased in the [File Transfers module](https://academy.hackthebox.com/course/preview/file-transfers)) and even compile some of the tools on your own using [Visual Studio](https://visualstudio.microsoft.com/downloads/).

Have fun, and don't forget to think outside the box!

\-mrb3n