
- **Windows** **Domain** is a group of users and computers under the administration of a given business
- The main idea behind a domain is to centralize the administration of common components of a Windows computer network in a single repository called **Active** **Directory** (**AD**). The server that runs the Active Directory services is known as a **Domain** **Controller** (**DC**)
- ![Windows Domain Overview](https://cdn-images.tryhackme.com/user-uploads/5ed5961c6276df568891c3ea/room-content/bebe5dfec0208bf563d01fa2dd1fb7a7.png)

- The main advantages of having a configured Windows Domain are:
	- Centralized identity management
	- Managing security policies.


### Active Directory
- The core of any Windows Domain is the **Active Directory Domain Service** (**AD DS**). This service acts as a catalogue that holds the information of all of the "objects" that exist on our network.
- Among the many object support by AD, we have users, groups, machines, printers, shares and many others.
- **Users**: 
	- The most common object type in Active Directory.
	- Users are one of the objects known as security principals, meaning that they can be authorized by the domain and can access resources like files or printers.
	- Users can be used to represent two types of entities:
		- **People** : users will generally represent persons in our organisation that need to access the network, like employees
		- **Services** : users like IIS (Internet Information Service) or MSSQL. They are different from regular users because they have only privileges to run their specific service.
- **Machines**:
	- Every computer joining the AD, a machine object will be created.
	- Also consider security principles.
	- Accounts are assigned just as any regular user. This account has somewhat limited rights within the domain itself.
	- They them-self are administrators on the assigned computer, and cannot be accessed by anyone except the computer itself.
	- **Note** : **Machine Account passwords are automatically rotated out and are generally comprised of 120 random characters.**
	- The machine account name is the computers name followed by a dollar sign. For example, machine named DC01 will have a machine account called DC01$.
- **Security** **Groups**: 
	- We can define user groups to assign rights to files or others resources to entire groups instead of a single user.
	- They are also consider security principles and can have privileges over resources on the network.
	- Groups can have both users and machines as members. If needed groups include other groups as well.

|                    |                                                                                                                                                           |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Security Group** | **Description**                                                                                                                                           |
| Domain Admins      | Users of this group have administrative privileges over the entire domain. By default, they can administer any computer on the domain, including the DCs. |
| Server Operators   | Users in this group can administer Domain Controllers. They cannot change any administrative group memberships.                                           |
| Backup Operators   | Users in this group are allowed to access any file, ignoring their permissions. They are used to perform backups of data on computers.                    |
| Account Operators  | Users in this group can create or modify other accounts in the domain.                                                                                    |
| Domain Users       | Includes all existing user accounts in the domain.                                                                                                        |
| Domain Computers   | Includes all existing computers in the domain.                                                                                                            |
| Domain Controllers | Includes all existing DCs on the domain.                                                                                                                  |

### Active Directory Users and Computers
- The object are organized in **Organisational** **Units** (**OU**) which are container of object that allow us to classify users and machines.
- OUs are mainly used to define sets of users with similar policing requirements. Like the people of the Sales department of our organization are likely to have a different set of policies applied than the people in IT.

You probably noticed already that there are other default containers apart from the THM OU. These containers are created by Windows automatically and contain the following:

- **Builtin:** Contains default groups available to any Windows host.
- **Computers:** Any machine joining the network will be put here by default. You can move them if needed.
- **Domain Controllers:** Default OU that contains the DCs in your network.
- **Users:** Default users and groups that apply to a domain-wide context.
- **Managed Service Accounts:** Holds accounts used by services in your Windows domain.

**Security Groups vs OUs**
- OUs are handy for applying policies to users and computers, which include specific configurations that pertain to sets of users depending on their particular role in the enterprise. **Remember, a user can only be a member of a single OU at a time.**
- Security Groups, are used to grant permissions over resources. For ex, we will use groups if we want to all some users to access a shared folder or network printer. A user can be a part of many groups, which is needed to grant access to multiple resources.

### Managing Users in AD
- Deleting extra OUs and users: If we try to delete a OU, AD by default has protection against accidental deletion so we have to enable the Advanced Features in the View menu. Right Click on OU -> Advanced Features -> Object -> Uncheck Protect Object from accidental deletion. Now try to delete the OU again. We will be prompted to confirm that we want to delete the OU, and as a result, any users, groups or OU under it will be deleted.
- **Delegation**: It is the process in which we can give specific users some control over some OUs and it allows us to grant users specific privileges to perform advanced tasks on OUs without needing a Domain Administrator to step in. To do so, we right-click on the OU -> Delegate Controls -> Add user or group -> Delegation of Control Wizard (Task to Delegate like Reset user password and force password change at next logon).
	- Now to use this powers we logon the privilege user but we cannot open the Active Directory Users and Group because we don't have privilege to open it.
	- Now we have to use the Powershell instead.
	- `Set-ADAccountPassword sophie -Reset -NewPassword (Read Host -AsSecureString -Prompt 'New Password') -Verbose` This will prompt us to enter a new password for user Sophie.
	- Since we wouldn't want Sophie to keep on using a password we know, we can also force a password reset at new logon with the following command: `Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose`

- **Managing Computer in AD**:
	- By default all the computer joining a domain will be put in the container called **Computers**. 
	- **Workstations**: Each user in the domain will likely be logging into a workstation. These devices should never have a privileged user signed into them.
	- **Servers**: Servers are generally use to provide services to users or others severs.
	- **Domain** **Controllers**: It allow us to manage the Active Directory Domain. They contain sensitive devices with the network as they contain hashed password of all users account with the environment.

- **Groups Policies**:
	- Windows manages such policies through **Group Policy Objects (GPO)**.
	- GPOs can contain policies aimed at either users or computers, allowing us to set a baseline on specific machines and identities.
	- To configure GPOs, we use Group Policy Management Tool.
	- To configure Group Policies, we first create a GPO under Group Policy Objects and then link it to the OU where we want the policy to apply.
	- We see image of 3 GPOs: **Default Domain Policy**, **Default Domain Control Policy**, **RDP Policy**. The Default Domain and RDP Policies are linked to the thm.local domain. This policies are applied to all sub-folder under the thm.local.
	- Examining inside Default Domain Policy, we see the first tab is **scope**.
	- The **Setting Tab** includes the actual contents of the GPO, each GPO has configurations that apply to computers only and configurations that apply to users only.

	![[Pasted image 20261023103643.png]]

	- Let's change the minimum password length policy to require users to have at least 10 characters in their password. To do this we Right-Click on the Default Domain Policy -> select Edit.
	- `Computer Configurations -> Policies -> Windows Setting -> Security Settings -> Account Policies -> Password Policy`
	- Double Click on Policies to learn more about them.
	- **GPO Distribution**: GPOs are distributed to the networks via a network share called `SYSVOL`, which is stored in the DC. All users in a domain should typically have access to this share over the network to sync their GPOs periodically. The **SYSVOL** share points by default to the `C:\Windows\SYSVOL\sysvol\` directory on each of the DCs in our network.
	- If we created and linked the GPOs, but for some reason, they still don't work, remember we can run `gpupdate /force` to force GPOs to be updated.

### Authentication Methods
- All credentials are stored in the Domain Controllers.
- Two protocols can be used for network authentication in windows domains:
	- **Kerberos**: Used by any recent version of Windows. This is the default protocol in any recent domain.
	- NetNTLM: Legacy authentication protocol kept for compatibility purposes.
- **Kerberos Authentication:**
	- Users that log into the service using Kerberos will be assigned tickets. Think of tickets as proof of a previous authentication.
	- The user sends their username and a timestamp encrypted using a key derived from their password to the **Key Distribution Center (KDC),** a service usually installed on the Domain Controller in charge of creating Kerberos tickets on the network.
	- The KDC will create and send back a **Ticket Granting Ticket (TGT)**, which will allow the user to request additional tickets to access specific services. Along with the TGT, a **Session Key** is given to the users which they will need to generate the following requests.
	- 
		![[Pasted image 20261024044622.png|418]]

	- Notice the TGT is encrypted using the **krbtgt** account's password hash, and therefore the user can't access its contents. Session Key is also a part of it, and the KDC has no need to store the Session Key as it can recover a copy by decrypting the TGT if needed.
	- When a user want to connect to a service, they will use this TGT to key the KDC for a **Ticket Granting Service (TGS)**. To request a TGS, the user will send their username and a timestamp encrypted using the Session Key, along with TGT and a **Service** **Principal** **Name** (**SPN**), which indicates the service and server name we intend to access.
	- As a result KDC send a TGS along with **Service** **Session** **Key**, which will authenticate to the service we want access for. The TGS is encrypted using a key derived from the **Service Owner Hash**. Service Owner is the user or machine account that the service runs under.
	- ![[Pasted image 20261024051148.png|421]]
	- The TGS can the be sent to the desired service to authenticate and establish a connection. The service will use its configured account's password hash to decrypt the TGS and validate the Service Session Key.
- **NetNTLM Authentication**:
	- NetNTLM works using a challenge-response mechanism.
	- The client sends a authentication request to the server they want to access.
	- The server generate a random number and sends it as a challenge to the client.
	- The client combines their NTLM password hash with the challenge to generate a response to the challenge and sends it back to the server for vertification.
	- The server forwards the challenge and the response to the DC for verification.
	- The DC uses the challenge to recalculate the response and if it match with the user response, the client is authenticated otherwise denied.
	- The server forward the authentication result to the client.
	- Note that the user's password is never transmitted through the network for security.

### Tree,Forest and Trust Relationship

- **Tree**:
	- Active Directory supports integrating multiple domains so that we can partition our network into units that can be managed independently. If we have two domain that share the same namespace (thm.local in our example), those domains can be joined into a Tree.
	- ![[Pasted image 20261024053758.png|346]]
	- Two subdomains called uk.thm.local and us.thm.local, each with its AD, computers and users.
	- This partitioned structure gives us better control over who can access what in the domain. The IT people from the UK will have their own DC that manages the UK resources only. For example, a UK user would not be able to manage US users. In that way, the Domain Administrators of each branch will have complete control over their respective DCs, but not other branches' DCs. Policies can also be configured independently for each domain in the tree.
	- A new security group needs to be introduced when talking about trees and forests. The **Enterprise Admins** group will grant a user administrative privileges over all of an enterprise's domains. Each domain would still have its Domain Admins with administrator privileges over their single domains and the Enterprise Admins who can control everything in the enterprise.
- **Forests**:
	- Suppose our company continues to grow and eventually acquires another company MHT Inc. 
	- The union of several trees with different namespace into the same network is known as a forest.
- **Trust Relationship**:
	- **One-way trust relationship**: In a one-way trust, if Domain AAA trust Domain BBB, this means that user on BBB can be authorized to access resources on AAA
	- **Two-way trust relationship:** It can also be made to allow both domains to mutually authorize users from the other.
	- It is important to note that having trust relationship between domains doesn't automatically grant access to all resources on other domains.