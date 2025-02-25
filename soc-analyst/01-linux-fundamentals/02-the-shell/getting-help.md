# Getting Help

---

Having established a solid foundation in Linux's structure, its various distributions, and the purpose of the shell, we're now prepared to put this knowledge into action. It's time to dive in, using commands directly in the terminal, as well as learning how to seek help when we encounter unfamiliar ones.

We will always stumble across tools whose optional parameters we do not know from memory or tools we have never seen before. Therefore it is vital to know how we can help ourselves to get familiar with those tools. The first two ways are the man pages and the help functions. It is always a good idea to familiarize ourselves with the tool we want to try first. We will also learn some possible tricks with some of the tools that we thought were not possible. In the man pages, we will find the detailed manuals with detailed explanations.

#### First Command:

Getting Help

```shell-session
swmpdnky@htb[/htb]$ ls

cacert.der  Documents  Music     Public     Videos
Desktop     Downloads  Pictures  Templates
```

The `ls` command in Linux and Unix systems is used to list the files and directories within the current folder or any specified directory, allowing you to see what's inside and manage files more effectively. Like most Linux commands, `ls` comes with additional options and features that help you filter or format the output to display exactly what you want. To discover which options a tool or command offers, there are several ways to get help. One such method is using the `man` command, which displays the manual pages for commands and provides detailed information about their usage.

#### Syntax:

Getting Help

```shell-session
swmpdnky@htb[/htb]$ man <tool>
```

Let us have a look at an example and get help for the `ls` command:

#### Example:

Getting Help

```shell-session
swmpdnky@htb[/htb]$ man ls
```

Getting Help

```shell-session
LS(1)                            User Commands                           LS(1)

NAME
       ls - list directory contents

SYNOPSIS
       ls [OPTION]... [FILE]...

DESCRIPTION
       List  information  about  the FILEs (the current directory by default).
       Sort entries alphabetically if none of -cftuvSUX nor --sort  is  speci‐
       fied.

       Mandatory  arguments  to  long  options are mandatory for short options
       too.

       -a, --all
              do not ignore entries starting with .

       -A, --almost-all
              do not list implied . and ..

       --author
 Manual page ls(1) line 1 (press h for help or q to quit)
```

After looking at some examples, we can also quickly look at the optional parameters without browsing through the complete documentation. We have several ways to do that.

#### Syntax:

Getting Help

```shell-session
swmpdnky@htb[/htb]$ <tool> --help
```

#### Example:

Getting Help

```shell-session
swmpdnky@htb[/htb]$ ls --help

Usage: ls [OPTION]... [FILE]...
List information about the FILEs (the current directory by default).
Sort entries alphabetically if none of -cftuvSUX nor --sort is specified.

Mandatory arguments to long options are mandatory for short options too.
  -a, --all                  do not ignore entries starting with .
  -A, --almost-all           do not list implied . and ..
      --author               with -l, print the author of each file
  -b, --escape               print C-style escapes for nongraphic characters
      --block-size=SIZE      with -l, scale sizes by SIZE when printing them;
                             e.g., '--block-size=M'; see SIZE format below

  -B, --ignore-backups       do not list implied entries ending with ~
  -c                         with -lt: sort by, and show, ctime (time of last
                             modification of file status information);
                             with -l: show ctime and sort by name;
                             otherwise: sort by ctime, newest first

  -C                         list entries by columns
<SNIP>
```

Some tools or commands like `curl` provide a short version of help by using `-h` instead of `--help`:

#### Syntax:

Getting Help

```shell-session
swmpdnky@htb[/htb]$ <tool> -h
```

#### Example:

Getting Help

```shell-session
swmpdnky@htb[/htb]$ curl -h

Usage: curl [options...] <url>
     --abstract-unix-socket <path> Connect via abstract Unix domain socket
     --anyauth       Pick any authentication method
 -a, --append        Append to target file when uploading
     --basic         Use HTTP Basic Authentication
     --cacert <file> CA certificate to verify peer against
     --capath <dir>  CA directory to verify peer against
 -E, --cert <certificate[:password]> Client certificate file and password
<SNIP>
```

As we can see, the results from each other do not differ in this example. Another tool that can be useful in the beginning is `apropos`. Each manual page has a short description available within it. This tool searches the descriptions for instances of a given keyword.

#### Syntax:

Getting Help

```shell-session
swmpdnky@htb[/htb]$ apropos <keyword>
```

#### Example:

Getting Help

```shell-session
swmpdnky@htb[/htb]$ apropos sudo

sudo (8)             - execute a command as another user
sudo.conf (5)        - configuration for sudo front end
sudo_plugin (8)      - Sudo Plugin API
sudo_root (8)        - How to run administrative commands
sudoedit (8)         - execute a command as another user
sudoers (5)          - default sudo security policy plugin
sudoreplay (8)       - replay sudo session logs
visudo (8)           - edit the sudoers file
```

Another useful resource to get help if we have issues to understand a long command is: [https://explainshell.com/](https://explainshell.com/)

Next, we'll be covering a large number of commands, many of which may be new to you. However, you now know how to seek help with any command you’re unfamiliar with, or unsure about its options. Also, we highly encourage you to explore your curiosity, taking as much time as needed to tinker and experiment with the tools presented. It will always be time well spent.