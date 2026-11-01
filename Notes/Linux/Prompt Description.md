
```
<username>@<hostname><current working directory>$
```

The home directory for a user is marked with a tilde <`~`> and is the default folder when we log in.

```
<username>@<hostname>[~]$
```

**The dollar sign, in this case, stands for a user.**
As soon as we log in as root, the character changes to a hash **#** and looks like this:

```
root@htb[/htb]#
```

For example, when we upload and run a shell on the target system, we may not see the username, hostname, and current directory. This may be due to the ==🟠**PSI**== variable in the **environment** not being set correctly.

**Unprivileged** - **User Shell Prompt**

```
$
```

**Privileged - Root Shell Prompt**

```
#
```

**The PS1 variable in Linux systems controls how our command prompt looks in the terminal. It's like a template that defines the text we see each time the system is ready for us to type a command. By customizing the PS1 variable, we can change the prompt to display information such as our username, our computer name, the current folder we are in or even add colors and special characters.**

**The prompt can be customized using special character and variables in the shell's configuration file (.bashrc for Bash Shell)**

