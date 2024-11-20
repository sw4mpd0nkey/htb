# Modules

As we mentioned previously, Metasploit `modules` are prepared scripts with a specific purpose and corresponding functions that have already been developed and tested in the wild. The `exploit` category consists of so-called proof-of-concept (`POCs`) that can be used to exploit existing vulnerabilities in a largely automated manner. Many people often think that the failure of the exploit disproves the existence of the suspected vulnerability. However, this is only proof that the Metasploit exploit does not work and not that the vulnerability does not exist. This is because many exploits require customization according to the target hosts to make the exploit work. Therefore, automated tools such as the Metasploit framework should only be considered a support tool and not a substitute for our manual skills.

Once we are in the `msfconsole`, we can select from an extensive list containing all the available Metasploit modules. Each of them is structured into folders, which will look like this:

## Syntax

```
<No.> <type>/<os>/<service>/<name>
```

### Example

```
794   exploit/windows/ftp/scriptftp_list
```

### Index No.

The `No.` tag will be displayed to select the exploit we want afterward during our searches. We will see how helpful the `No.` tag can be to select specific Metasploit modules later.

### Type

The `Type` tag is the first level of segregation between the Metasploit `modules`. Looking at this field, we can tell what the piece of code for this module will accomplish. Some of these `types` are not directly usable as an `exploit` module would be, for example. However, they are set to introduce the structure alongside the interactable ones for better modularization. To explain better, here are the possible types that could appear in this field:

| **Type** | **Description** |
| --- | --- |
| `Auxiliary` | Scanning, fuzzing, sniffing, and admin capabilities. Offer extra assistance and functionality. |
| `Encoders` | Ensure that payloads are intact to their destination. |
| `Exploits` | Defined as modules that exploit a vulnerability that will allow for the payload delivery. |
| `NOPs` | (No Operation code) Keep the payload sizes consistent across exploit attempts. |
| `Payloads` | Code runs remotely and calls back to the attacker machine to establish a connection (or shell). |
| `Plugins` | Additional scripts can be integrated within an assessment with `msfconsole` and coexist. |
| `Post` | Wide array of modules to gather information, pivot deeper, etc. |

Note that when selecting a module to use for payload delivery, the `use <no.>` command can only be used with the following modules that can be used as `initiators` (or interactable modules):

| **Type** | **Description** |
| --- | --- |
| `Auxiliary` | Scanning, fuzzing, sniffing, and admin capabilities. Offer extra assistance and functionality. |
| `Exploits` | Defined as modules that exploit a vulnerability that will allow for the payload delivery. |
| `Post` | Wide array of modules to gather information, pivot deeper, etc. |

### OS 

The `OS` tag specifies which operating system and architecture the module was created for. Naturally, different operating systems require different code to be run to get the desired results.

### Service

The `Service` tag refers to the vulnerable service that is running on the target machine. For some modules, such as the `auxiliary` or `post` ones, this tag can refer to a more general activity such as `gather`, referring to the gathering of credentials, for example.

### Name

Finally, the `Name` tag explains the actual action that can be performed using this module created for a specific purpose.

## Searching for Modules 