In this room, we will cover a privilege escalation attack that abuses **delegated Managed Service Account** (dMSA) to succeed any account, provided that certain conditions are met. This attack was discovered by Yuval Gordon from Akamai and published in the Security Research blog, [BadSuccessor: Abusing dMSA to Escalate Privileges in Active Directory](https://www.akamai.com/blog/security-research/abusing-dmsa-for-privilege-escalation-in-active-directory?). In simple terms, this attack allows a user who can control a dMSA object to achieve domain admin access.

Active Directory (AD) has many types of account, including user, computer, and group accounts. Among the user accounts, we have service accounts of various types: traditional service accounts and managed service accounts.

A Managed Service Account (MSA) is a special type of account in AD that allows running services or scheduled tasks on MS Windows systems; their key features is that they don't require human user to manage the password. There are three types:

- **Standalone Managed Service Account (sMSA)** is designed for a service running on a single computer. The AD handles the account password, including rotating it every 30 days by default. This service account was introduced in Windows Server 2008 R2
- **Group Managed Service Account (gMSA)** is designed for a service running on a multiple computers or servers. Similarly, the AD handles the account password. This services account was introduced in Windows Server 2012.
- **Delegated Managed Service Account (dMSA)** is the most recent addition to the family of service accounts. It enables the migration of a legacy service account to a machine account i.e., from a account  that is not MSA to dMSA. Unlike a gMSA that is managed by AD and allows running a service on multiple servers, a dMSA is managed by a an administrator and allows running a service on a specific server. Introduced in 2025.

### Reconnaissance
- We have credentials of an IT personal Terry Byte.
- Terry runs a script Get-BadSuccessorOUPermissions.ps1 to identity accounts that can create dMSA in their organization units. It searches for accounts that have certain privileges `$relevantRights = "CreateChild|GenericAll|WriteDACL|WriteOwner"`

### Exploitation Using Windows
- First we create a dMSA in an OU that the user has write access to and then by modifying the dMSA object attribute to mimic a successful migration.
- Once this two step are completed getting a Ticket Granting Ticket and proceeding to get credentials would be possible.
- **SharpSuccessor** written in C# and requires compilation. We complied it along with Rubeus.
- `.\SharpSuccessor.exe add /path:"ou=LabOU,dc=tryhackme,dc=local" /account:tbyte /name:pentent_dmsa /impersonate:Administrator`
- In the next step we can request a TGT. Using Rubeus with the command `.\Rubeus.exe tgtdeleg /nowrap`. The `tgtdeleg` abuses a lesser-known feature of Kerberos's unconstrained delegation. It asks the system to impersonate the current user and export their TGT directly from memory via a legitimate API call. On the other hand, the /nowrap option outputs the result in a single line, perfect for copy-paste reuse without formatting headaches.
- With the TGT claimed using tgtdeleg, we can use it to impersonate the dMSA account to request a Ticket Granting service with the following command `.\Rubeus.exe asktgs /targetuser:pentest_dmsa$ /service:krbtgt/tryhackme.local /opsec /dmsa /nowrap /ptt /ticket:dt...`