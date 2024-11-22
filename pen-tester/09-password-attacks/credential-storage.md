# Credential Storage

Every application that supports authentication mechanisms compares the given entries/credentials with local or remote databases. In the case of local databases, these credentials are stored locally on the system. Web applications are often vulnerable to SQL injections, which can lead to the worst-case scenario where the attackers view the entirety of an organization's data in plain text.

There are many different wordlists that contain the most commonly used passwords. An example of one of these lists is `rockyou.txt`. This list includes about 14 million unique passwords, and it was created after a data breach of the company RockYou, which contained a total of 32 million user accounts. The RockYou company stored all the credentials in plain text in their database, which the attackers could view. after a successful SQL injection attack.

We also know that every operating system supports these types of authentication mechanisms. The stored credentials are therefore stored locally. Let's look at how these are created, stored, and managed by Windows and Linux-based systems in more detail.

## Linux

As we already know, Linux-based systems handle everything in the form of a file. Accordingly, passwords are also stored encrypted in a file. This file is called the `shadow` file and is located in `/etc/shadow` and is part of the Linux user management system. In addition, these passwords are commonly stored in the form of `hashes`. An example can look like this:

