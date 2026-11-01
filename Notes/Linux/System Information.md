
|**Command**|**Description**|
|---|---|
|`whoami`|Displays current username.|
|`id`|Returns users identity|
|`hostname`|Sets or prints the name of current host system.|
|`uname`|Prints basic information about the operating system name and system hardware.|
|`pwd`|Returns working directory name.|
|`ifconfig`|The ifconfig utility is used to assign or to view an address to a network interface and/or configure network interface parameters.|
|`ip`|Ip is a utility to show or manipulate routing, network devices, interfaces and tunnels.|
|`netstat`|Shows network status.|
|`ss`|Another utility to investigate sockets.|
|`ps`|Shows process status.|
|`who`|Displays who is logged in.|
|`env`|Prints environment or sets and executes command.|
|`lsblk`|Lists block devices.|
|`lsusb`|Lists USB devices|
|`lsof`|Lists opened files.|
|`lspci`|Lists PCI devices.|

### Id
- Expands on the `whoami` command and prints out our effective group membership and IDs
- This can be of interest to penetration testers looking to see what access a user may have.
- For example:

```
cry0l1t3@htb[/htb]$ id

uid=1000(cry0l1t3) gid=1000(cry0l1t3) groups=1000(cry0l1t3),1337(hackthebox),4(adm),24(cdrom),27(sudo),30(di```
p),46(plugdev),116(lpadmin),126(sambashare)
```


### Find
- Beside the function to find files and folders, this tool also contains the function to filter the results.

For example,

```
Crackhead2001@htb[/htb]$ find / -type f -name *.conf -user root -size +20k -newermt 2020-03-03 -exec ls -al {} \; 2>/dev/null
```


| **Option**            | **Description**                                                                                                                                                                                                                                                                |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `-type f`             | Hereby, we define the type of the searched object. In this case, '`f`' stands for '`file`'.                                                                                                                                                                                    |
| `-name *.conf`        | With '`-name`', we indicate the name of the file we are looking for. The asterisk (`*`) stands for 'all' files with the '`.conf`' extension.                                                                                                                                   |
| `-user root`          | This option filters all files whose owner is the root user.                                                                                                                                                                                                                    |
| `-size +20k`          | We can then filter all the located files and specify that we only want to see the files that are larger than 20 KiB.                                                                                                                                                           |
| `-newermt 2020-03-03` | With this option, we set the date. Only files newer than the specified date will be presented.                                                                                                                                                                                 |
| `-exec ls -al {} \;`  | This option executes the specified command, using the curly brackets as placeholders for each result. The backslash escapes the next character from being interpreted by the shell because otherwise, the semicolon would terminate the command and not reach the redirection. |
| `2>/dev/null`         | This is a `STDERR` redirection to the '`null device`', which we will come back to in the next section. This redirection ensures that no errors are displayed in the terminal. This redirection must `not` be an option of the 'find' command.                                  |