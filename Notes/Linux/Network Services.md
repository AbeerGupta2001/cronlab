### SSH (Secure Shell)
- It is a network protocol that allows the secure transmission of data and commands over a network.
- The widely used SSH server is the **OpenSSH server**. 
- Administrators use OpenSSH to securely manage remote systems by establishing an encrypted connection to remote host.
- Install OpenSSH - `sudo apt install openssh-server -y`
- Status Check - `systemctl status ssh`
- OpenSSH can be configured and customized by editing the file `/etc/ssh/sshd_config`. Here we can adjust settings such as the maximum number of concurrent connections, the use of passwords or keys for logins, host key checking, and more.

### NFS (Network File System)
- It is a network protocol that allows us to store and manage file on remote systems as if they were stored on the local system.
- For Linux there are several NFS servers including **NFS**-**UTILS** (Ubuntu), **NFS**-**Ganesha** (Solaris) and **OpenNFS** (Redhat Linux)
- Install - `sudo apt install nfs-kernel-server -y`
- Server Status - `systemctl status nfs-kernel-server`
- We can configure NFS via the configuration file `/etc/exports`. This file specifies which directories should be shared and the access rights for users and systems. It is also possible to configure settings such as the transfer speed and the use of encryption.

| Permissions      | Descriptions                                                                                                                                               |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `rw`             | Gives users and systems read and write permissions to the shared directory.                                                                                |
| `ro`             | Gives users and systems read-only access to the shared directory.                                                                                          |
| `no_root_squash` | Prevents the root user on the client from being restricted to the rights of a normal user.                                                                 |
| `root_squash`    | Restricts the rights of the root user on the client to the rights of a normal user.                                                                        |
| `sync`           | Synchronizes the transfer of data to ensure that changes are only transferred after they have been saved on the file system.                               |
| `async`          | Transfers data asynchronously, which makes the transfer faster, but may cause inconsistencies in the file system if changes have not been fully committed. |
**Create NFS Share**
```
cry0l1t3@htb:~$ mkdir nfs_sharing
cry0l1t3@htb:~$ echo '/home/cry0l1t3/nfs_sharing hostname(rw,sync,no_root_squash)' >> /etc/exports
cry0l1t3@htb:~$ cat /etc/exports | grep -v "#"

/home/cry0l1t3/nfs_sharing hostname(rw,sync,no_root_squash)
```

- If we have created an NFS share and want to work with it on the target system, we have to mount it first.
**Mount NFS Share**

```
cry0l1t3@htb:~$ mkdir ~/target_nfs
cry0l1t3@htb:~$ mount 10.129.12.17:/home/john/dev_scripts ~/target_nfs
cry0l1t3@htb:~$ tree ~/target_nfs

target_nfs/
├── css.css
├── html.html
├── javascript.js
├── php.php
└── xml.xml

0 directories, 5 files
```

### Web server
- A web server is a software that delivers data, documents, applications, and various function over the Internet.
- It utilizes the HyperText Transfer Protocol (HTTP) to transmit data to client such as web browsers and to receive requests from these client.
- **Install Apache Web Server** - `sudo apt install apache2 -y`
- For Apache2, to specify which folder can be accessed, we can edit the file `/etc/apache2/apache2.conf` with a text editor.

**Apache Configuration**
```
<Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
</directory>
```

- This session specifies that the default `/var/www/html` folder is accessible, that users can use the `Indexes and FollowSymLinks` options, that changes to files in this directory can be overridden with `AllowOverride All`, and that `Require all granted` grants all users access to this directory.  
- It is also possible to customize individual setting at the directory level by using the `.htaccess` file, which we can create in the directory in question. This file allows us to configure certain directory-level settings, such as access controls, without having to customize the Apache configuration file.
- We can also add modules to get features like `mod_rewrite`, `mod_security`, and `mod_ssl` that help us improve the security of our web application.
- **Install Python & Web server** - `sudo apt install python3 -y` and `python3 -m http.server`. This starts the web server on the `TCP/8000`
	- `python3 -m http.server --directory /home/abeer/target_files` This will start a python web server on the `TCP/8000` and we access the `/home/abeer/target_files`

### VPN (Virtual Private Network)

- VPN functions like a secure, invisible tunnel that connects us to another network, allowing seamless and protected access as if we were physically present within it.
- VPN serve to anonymize internet traffic and block external intrusions,further [bolstering](obsidian://open?vault=Notes&file=Meaning) security. 
- **OpenVPN** stands out as a popular open source option compatible with various OS.
- **Install OpenVPN** - `sudo apt install openvpn -y`
- OpenVPN can be customized and configured by editing the configuration file `/etc/openvpn/server.conf`
- If we want to connect to an OpenVPN server, we can use the `.ovpn` file we received from the server and save it on our system. `sudo openvpn --config internal.ovpn`