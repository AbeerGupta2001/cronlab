
## Network Access Control

Network access control (NAC) is a crucial component of network security, especially in today's era of increasing cyber threats. As a penetration tester, it is vital to understand the significance of NAC in protecting the network and the various NAC technologies that can be utilized to enhance security measures. NAC is a security system that ensures that only authorized and compliant devices are granted access to the network, preventing unauthorized access, data breaches, and other security threats. By implementing NAC, organizations can be confident in their ability to protect their assets and data from cybercriminals who always seek to exploit system vulnerabilities. The following are the different NAC technologies that can be used to enhance security measures:

- Discretionary access control (DAC)
- Mandatory access control (MAC)
- Role-based access control (RBAC)

These technologies are designed to provide different levels of access control and security. Each technology has its unique characteristics and is suitable for different use cases. As a penetration tester, it is essential to understand these technologies and their specific use cases to test and evaluate the network's security effectively.

#### Discretionary Access Control

DAC is a crucial component of modern security systems as it helps organizations provide access to their resources while managing the associated risks of unauthorized access. It is a widely used access control system that enables users to manage access to their resources by granting resource owners the responsibility of controlling access permissions to their resources. This means that users and groups who own a specific resource can decide who has access to their resources and what actions they are authorized to perform. These permissions can be set for reading, writing, executing, or deleting the resource.

#### Mandatory Access Control

MAC is used in infrastructure that provides more fine-grained control over resource access than DAC systems. Those systems define rules that determine resource access based on the resource's security level and the user's security level or process requesting access. Each resource is assigned a security label that identifies its security level, and each user or process is assigned a security clearance that identifies its security level. Access to a resource is only granted if the user's or process's security level is equal to or greater than the security level of the resource. MAC is often used in operating systems and applications that require a high level of security, such as military or government systems, financial systems, and healthcare systems. MAC systems are designed to prevent unauthorized access to resources and minimize the impact of security breaches.

#### Role-based Access Control

RBAC assigns permissions to users based on their roles within an organization. Users are assigned roles based on their job responsibilities or other criteria, and each role is granted a set of permissions that determine the actions they can perform. RBAC simplifies the management of access permissions, reduces the risk of errors, and ensures that users can access only the resources necessary to perform their job functions. It can restrict access to sensitive resources and data, limit the impact of security breaches, and ensure compliance with regulatory requirements. Compared to Discretionary Access Control (DAC) systems, RBAC provides a more flexible and scalable approach to managing resource access. In an RBAC system, each user is assigned one or more roles, and each role is assigned a set of permissions that define the user's actions. Resource access is granted based on the user's assigned role rather than their identity or ownership of the resource. RBAC systems are typically used in environments with many users and resources, such as large organizations, government agencies, and financial institutions.

## Hardening

Several mechanisms are highly effective in securing Linux systems in keeping our and other companies' data safe. Three such mechanisms are SELinux, AppArmor, and TCP wrappers. These tools are designed to safeguard Linux systems against various security threats, from unauthorized access to malicious attacks. This is critical not only during penetration tests, where systems are intentionally stressed to uncover vulnerabilities, but also in real-world scenarios where an actual compromise could have serious consequences (few situations are as severe as a real-life breach.) By implementing these security measures and ensuring that we set up corresponding protection against potential attackers, we can significantly reduce the risk of data leaks and ensure our systems remain secure. While these tools share some similarities, they also have important differences.

#### Security-Enhanced Linux

Security-Enhanced Linux (`SELinux`) is a mandatory access control (`MAC`) system integrated into the Linux kernel. It provides fine-grained control over access to system resources and applications by enforcing security policies. These policies define the permissions for each process and file on the system, significantly limiting the damage that a compromised process or service can do. SELinux operates at a low level, and though it offers strong security, it can be complex to configure and manage due to its granular controls.

#### AppArmor

Like SELinux, `AppArmor` is a MAC system that controls access to system resources and applications, but it operates in a simpler, more user-friendly manner. AppArmor is implemented as a Linux Security Module (`LSM`) and uses application profiles to define what resources an application can access. While it may not provide the same level of fine-grained control as SELinux, AppArmor is often easier to configure and is generally considered more straightforward for day-to-day use.

#### TCP Wrappers

`TCP wrappers` are a host-based network access control tool that restricts access to network services based on the IP address of incoming connections. When a network request is made, TCP wrappers intercept it, checking the request against a list of allowed or denied IP addresses. This is a simple yet effective way to control access to services, especially for blocking unauthorized systems from accessing networked resources. While it does not offer the fine-grained control of SELinux or AppArmor, TCP wrappers are an excellent tool for basic network-level protection.

Regarding similarities, the three security mechanisms share the common goal of ensuring the safety and security of Linux systems. In addition to providing extra protection, they can restrict access to resources and services, thus reducing the risk of unauthorized access and data breaches. It's also worth noting that these mechanisms are readily available as part of most Linux distributions, making them accessible to us to enhance their systems' security. Furthermore, these mechanisms can be easily customized and configured using standard tools and utilities, making them a convenient choice for Linux users.

Although both `SELinux` and `AppArmor` are MAC systems that provide fine-grained control, they work in different ways. SELinux is deeply integrated into the kernel and offers more detailed security controls, but it can be more complex to configure and maintain. In contrast, AppArmor operates as a kernel module and uses profile-based security, making it easier to manage, though it may not offer the same level of granularity as SELinux.

On the other hand, `TCP wrappers` focus on controlling access to network services based on client IP addresses, which makes it simpler but limited to network-level access control. It doesn't offer the broader system resource protections that SELinux and AppArmor provide, but it’s useful for restricting access to services from unauthorized systems.

---

## Setting Up

As we navigate the world of Linux, we inevitably encounter a wide range of technologies, applications, and services that we need to become familiar with. This is a crucial skill, particularly if we work in cybersecurity and strive to improve our expertise continuously. For this reason, we highly recommend dedicating time to learning about configuring important security measures such as `SELinux`, `AppArmor`, and `TCP wrappers` on your own. By taking on this (optional but highly efficient) challenge, you'll deepen your understanding of these technologies, build up your problem-solving skills, and gain valuable experience that will serve you well in the future. We highly recommend to use a personal VM and make snapshots before making changes.

When it comes to implementing cybersecurity measures, there is no one-size-fits-all approach. It is important to consider the specific information you want to protect and the tools you will use to do so. However, you can practice and implement several optional tasks with others in the Discord channel to increase your knowledge and skills in this area. By taking advantage of the helpfulness of others and sharing your own expertise, you can deepen your understanding of cybersecurity and help others do the same. Remember, explaining concepts to others is essential to teaching and learning.

#### SELinux

|||
|---|---|
|1.|Install SELinux on your VM.|
|2.|Configure SELinux to prevent a user from accessing a specific file.|
|3.|Configure SELinux to allow a single user to access a specific network service but deny access to all others.|
|4.|Configure SELinux to deny access to a specific user or group for a specific network service.|

#### AppArmor

|||
|---|---|
|5.|Configure AppArmor to prevent a user from accessing a specific file.|
|6.|Configure AppArmor to allow a single user to access a specific network service but deny access to all others.|
|7.|Configure AppArmor to deny access to a specific user or group for a specific network service.|

#### TCP Wrappers

|||
|---|---|
|8.|Configure TCP wrappers to allow access to a specific network service from a specific IP address.|
|9.|Configure TCP wrappers to deny access to a specific network service from a specific IP address.|
|10.|Configure TCP wrappers to allow access to a specific network service from a range of IP addresses.|
