
### Cut
- Mainly used for content filtering
- Specific results with different characters may be separated as delimiters.

For example 
```
	cat /etc/password | grep -v "false \| nologin" | cut -d":" -f1
	
	root
	sync
	postgres
	mrb3n
	cry0l1t3
	htb-student
```

### Column
Since search result can often have a unclear representation, the tool `column` is well suited to display such results in tabular form using the `-t`. 

```
Crackhead2001@htb[/htb]$ cat /etc/passwd | grep -v "false\|nologin" | tr ":" " " | column -t

root         x  0     0      root               /root                /bin/bash
sync         x  4     65534  sync               /bin                 /bin/sync
postgres     x  111   117    PostgreSQL         administrator,,,    /var/lib/postgresql     /bin/bash
mrb3n        x  1000  1000   mrb3n              /home/mrb3n          /bin/bash
cry0l1t3     x  1001  1001   /home/cry0l1t3     /bin/bash
htb-student  x  1002  1002   /home/htb-student  /bin/bash
```

## Awk

As we may have noticed, the line for the user "`postgres`" has one column too many. To keep it as simple as possible to sort out such results, the (`g`)`awk` programming is beneficial, which allows us to display the first (`$1`) and last (`$NF`) result of the line.

```
Crackhead2001@htb[/htb]$ cat /etc/passwd | grep -v "false\|nologin" | tr ":" " " | awk '{print $1, $NF}'

root /bin/bash
sync /bin/sync
postgres /bin/bash
mrb3n /bin/bash
cry0l1t3 /bin/bash
htb-student /bin/bash
```

### Sed

There will come moments when we want to change specific names in the whole life or standard input. 
Sed is a stream editor used for substituting text.
Sed looks for patterns we have defined in the form of regular expression and replaces them with another pattern that we have also defined. 
For example
```
Crackhead2001@htb[/htb]$ cat /etc/passwd | grep -v "false\|nologin" | tr ":" " " | awk '{print $1, $NF}' | sed 's/bin/HTB/g'

root /HTB/bash
sync /HTB/sync
postgres /HTB/bash
mrb3n /HTB/bash
cry0l1t3 /HTB/bash
htb-student /HTB/bash
```
The "`s`" flag at the beginning stands for the substitute command. Then we specify the pattern we want to replace. After the slash (`/`), we enter the pattern we want to use as a replacement in the third position. Finally, we use the "`g`" flag, which stands for replacing all matches.


### Regular Expressions

**Grouping**

### Grouping Operators

|     | Operators | Description                                                                                                                                                                 |
| --- | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | `(a)`     | The round brackets are used to group parts of a regex. Within the brackets, you can define further patterns which should be processed together.                             |
| 2   | `[a-z]`   | The square brackets are used to define character classes. Inside the brackets, you can specify a list of characters to search for.                                          |
| 3   | `{1,10}`  | The curly brackets are used to define quantifiers. Inside the brackets, you can specify a number or a range that indicates how often a previous pattern should be repeated. |
| 4   | \|        | Also called the OR operator and shows results when one of the two expressions matches                                                                                       |
| 5   | `.*`      | Operates similarly to an AND operator by displaying results only when both expressions are present and match in the specified order                                         |

### Change Owner
- To change the owner and/or the group assignments of a file or directory, we can use the chown command. 

**Syntax**
```
cry0l1t3@htb[/htb]$ chown <user>:<group> <file/directory>
```
Example
```
cry0l1t3@htb[/htb]$ chown root:root shell && ls -l shell

-rwxr-xr--   1 root root 0 May  4 22:12 shell
```

### SUID & SGID (This permission are on files)

- Special permission on files through the Set User ID (`SUID`) and Set Group ID (`SGID`) bits. These bits function like temporary access passes, enabling users to run certain programs with the privileges of another user or group. For example, administrators can use SUID or SGID to grant users elevated rights for specific applications, allowing tasks to be performed with the necessary permissions, even if the user themselves doesn't normally have them. 
- The presence of these permissions is indicated by an `s` in place of the usual `x` in the file's permission set. When a program with SUID or SGID bit set is executed, it runs with the permissions of the file's owner or group, rather than the user who launched it.
- SUID allows executables to run with the file owner's privileges instead of the user executing it
- SGID allows files runs executables with the file's group privileges; on directories, new files inherit the directory's group
- Sticky Bits on directories restricts file deletion to file owner, directory owner, or root only.
- Only file owner or root can set SUID/SGID bits
- Octual values: 4 = SUID, 2 SGID, 1 sticky bits
- Sticky bits in Linux are like locks on files within shared spaces. When set on a directory, the sticky bit adds an extra layer of security, ensuring that only certain individuals can modify or delete files, even if others have access to the directory.
Example for Sticky bit
```
cry0l1t3@htb[/htb]$ ls -l

drw-rw-r-t 3 cry0l1t3 cry0l1t3   4096 Jan 12 12:30 scripts
drw-rw-r-T 3 cry0l1t3 cry0l1t3   4096 Jan 12 12:32 reports
```
In this example, we see that both directories have the sticky bit set. However the reports folder have a uppercase T and the scripts folder has a lowercase t.

If the sticky bit is capitalized (`T`), then this means that all other users do not have execute (`x`) permissions and therefore, cannot see the contents of the folder nor run any programs from it. The lowercase sticky bit (`t`) is the sticky bit where the `execute (x)` permissions have been set.

### APT (Advanced Package Manager)

- The "main" repository folder is `/etc/apt/sources.list`
- APT uses a database called APT cache. This is used to provide information about packages installed on our system offline. We can search the APT cache. `apt-cache search impacket`
- List all install packages `apt list --installed`

We can also use `systemctl` to list all service

```
systemctl list-units --type=service
```

It is quite possible that the service do not start due to an error. To see the problem, we can use the tool `journalctl` to view the logs.

```
journalctl -u ssh.service --no-pager
```

### User-Installed Services
- These services are added by users and typically include server application and other background processes that provide specific features and capabilities.
- While not critical for the system to operate, they enhance functionally and provide additional features based on the user's preferences.
- Daemons (background services in Linux) are often identified by the letter `d` at the end of their program names, such as sshd (SSH daemon) or systemd. 
- In general, there are just a few goals that we have when we deal with a service or a process.
	- Start / Restart a service / process
	- Stop a service / process
	- See what is / was happening with a service / process
	- Enable / Disable a service / process on boot
	- Find a service / process
- Most modern Linux distros have adopted systemd as their initialization system (init system). It is the first process that start during the boot process and is assign the Process ID (PID)
- All processes are assigned a **PID** and can be viewed user **/proc/** directory, which contains information about each process.
- Processes also have a Parent Process ID (**PPID**), indicating that they were started by another process, making them child processes.


### Systemctl
- Starting a service like ssh `systemctl start ssh`
- Checking status `systemctl status ssh`
- Enable service means run this service after startup `systemctl enable ssh`. This add OpenSSH to the SysV script.
- Checking process `ps -aux | grep ssh`
- List all services `systemctl list-units --type=service`
- Check error related to a service `journalctl -u ssh.service --no-pager`

### Kill a Process
- A process can be in the following states: **Running**, **Waiting**, **Stopped**, **Zombie**.
- Processes can be controlled using **kill**, **pkill**, **pgrep**, and **killall**.
- View all signals with `kill -l` command.

The most commonly used signals are:

| **Signal** | **Description**                                                                                                          |
| ---------- | ------------------------------------------------------------------------------------------------------------------------ |
| `1`        | `SIGHUP` - This is sent to a process when the terminal that controls it is closed.                                       |
| `2`        | `SIGINT` - Sent when a user presses `[Ctrl] + C` in the controlling terminal to interrupt a process.                     |
| `3`        | `SIGQUIT` - Sent when a user presses `[Ctrl] + D` to quit.                                                               |
| `9`        | `SIGKILL` - Immediately kill a process with no clean-up operations.                                                      |
| `15`       | `SIGTERM` - Program termination.                                                                                         |
| `19`       | `SIGSTOP` - Stop the program. It cannot be handled anymore.                                                              |
| `20`       | `SIGTSTP` - Sent when a user presses `[Ctrl] + Z` to request for a service to suspend. The user can handle it afterward. |
 - To kill a process `kill 9 <PID>`

### Background a Process
- Sometimes it will be necessary to put the scan or process we just started in the background to continue using the current session to interact with the system or start other processes
- We can do with the shortcut `[CTRL + Z]`
- We can also use `&` at end of the command to background a process.
For examples,
```
Crackhead2001@htb[/htb]$ ping -c 10 www.hackthebox.eu > output.log 2>&1 & 
```
- All background processes can be displayed with the following command `jobs`

### Foreground a Process

- `jobs` command to see the background processes.

```
Crackhead2001@htb[/htb]$ jobs

[1]+  Running                 ping -c 10 www.hackthebox.eu &
```

- If we want to get the background process into the foreground and interact with it again we can use the `fg <ID>` command.
```
Crackhead2001@htb[/htb]$ fg 1
ping -c 10 www.hackthebox.eu

--- www.hackthebox.eu ping statistics ---
10 packets transmitted, 0 received, 100% packet loss, time 9206ms
```

### CURL
- cURL is a tool that allows us to transfer files from the shell over protocols like HTTP, HTTPS, FTP, SFTP, FTPS, or SCP.