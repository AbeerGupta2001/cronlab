
## Where TLS Fits in the Network Model

The common protocols covered so far send data in cleartext, making it possible for anyone with access to the network to capture, save, and analyse the exchanged messages. The image below shows the ISO/OSI network layers. The protocols covered so far in this room are on the application layer. Encryption can be added via the presentation layer. As a result, data will be presented in an encrypted format (ciphertext) instead of its original form.

![](https://cdn-images.tryhackme.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/645b19f5d5848d004ab9c9e2-1772442556420.png)

It is worth noting that the OSI model is a conceptual simplification. In practice, TLS operates between the transport and application layers rather than mapping neatly to a single OSI layer. However, the diagram is useful for understanding that encryption sits below the application protocol, wrapping its data before transmission.

**Upgrading Protocols with TLS**

An existing cleartext protocol can be upgraded to use encryption via TLS. The following table lists the protocols covered so far and their default ports before and after the encryption upgrade via TLS.

|Protocol|Default Port|Secured Protocol|Default Port with TLS|
|---|---|---|---|
|HTTP|80|HTTPS|443|
|FTP|21|FTPS|990|
|SMTP|25|SMTPS|465|
|POP3|110|POP3S|995|
|IMAP|143|IMAPS|993|

TLS is not limited to web and email protocols. DNS can also be secured using TLS. **DNS over TLS (DoT)** encrypts DNS queries by wrapping standard DNS traffic inside a TLS connection, typically on port 853. A related approach, **DNS over HTTPS (DoH)**, sends DNS queries as HTTPS requests on port 443. Both prevent eavesdropping on DNS lookups, which would otherwise reveal which websites a user is visiting.

## Implicit TLS vs STARTTLS

There are two approaches to adding TLS to a protocol:

**Implicit TLS** uses a dedicated port (as shown in the table above). The connection is encrypted from the start. When you connect to port 443 for HTTPS or port 993 for IMAPS, TLS negotiation begins immediately.

**STARTTLS** allows upgrading an existing cleartext connection to TLS on the same port. The client connects on the standard port (e.g., port 25 for SMTP), and then issues a `STARTTLS` command to upgrade the connection to TLS. This approach is common for email protocols. For SMTP, port 587 (submission) with STARTTLS is the recommended configuration for mail clients.

Both approaches provide encryption. However, implicit TLS is generally preferred because STARTTLS can be vulnerable to downgrade attacks if not properly implemented. An attacker performing a MITM attack could strip the `STARTTLS` command from the communication, forcing the connection to remain in cleartext.

## How HTTPS Works

Consider the case of HTTP. Initially, to retrieve a web page over HTTP, the web browser would need to perform at least the following two steps:

1. Establish a TCP connection with the remote web server.
2. Send HTTP requests to the web server, such as `GET` and `POST` requests.

HTTPS requires an additional step to encrypt the traffic. The new step takes place after establishing a TCP connection and before sending HTTP requests. As a result, HTTPS requires at least the following three steps:

1. Establish a TCP connection.
2. **Establish a TLS connection.**
3. Send HTTP requests to the web server.

## The TLS Handshake

To establish a TLS connection, the client needs to perform a handshake with the server. The handshake differs between TLS versions. Here is a simplified overview of the TLS 1.2 handshake:

![](https://cdn-images.tryhackme.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/645b19f5d5848d004ab9c9e2-1773222504290.png)

1. **ClientHello**: The client sends a message indicating its capabilities, including supported TLS versions, cipher suites, and a random value.
2. **ServerHello**: The server responds with the selected connection parameters, its certificate (a digital file signed by a Certificate Authority to prove the server's identity), and its own random value.
3. **Key Exchange**: The client and server exchange information needed to generate the shared secret key. The exact process depends on the chosen cipher suite.
4. **Finished**: Both sides confirm the handshake completed successfully and switch to encrypted communication.

**TLS 1.3 Improvements**

TLS 1.3 significantly improves upon earlier versions:

- **Faster handshake**: TLS 1.3 requires only one round trip (1-RTT) to establish a connection, compared to two round trips for TLS 1.2. It even supports 0-RTT resumption for returning clients, though this has some security trade-offs.
- **Forward secrecy by default**: All TLS 1.3 cipher suites provide forward secrecy, meaning that if the server's private key is compromised in the future, past recorded sessions cannot be decrypted.
- **Simplified cipher suites**: Outdated and insecure algorithms have been removed. There are no more choices that could lead to weak configurations.
- **Encrypted handshake**: More of the handshake is encrypted, revealing less information to observers.

The key point is this: the client and server securely agree on a secret key that a third party monitoring the channel cannot discover. All further communication is encrypted using this key. As a result, once a TLS handshake has been established, HTTP requests and exchanged data are not accessible to anyone watching the communication channel.

## Certificates and Trust

For TLS to be effective, especially when browsing the web over HTTPS, the system relies on public certificates signed by Certificate Authorities (CAs) trusted by your system. When you browse to [TryHackMe](https://tryhackme.com/) over HTTPS, your browser expects the TryHackMe web server to provide a signed certificate from a trusted certificate authority. This ensures that you are communicating with the correct server and that a MITM attack cannot easily occur.

![](https://cdn-images.tryhackme.com/user-uploads/5f04259cf9bf5b57aed2c476/room-content/2362c08e3a718863a1b1b56279931538.png)

In the figure above, the following information is visible:

1. **To whom is the certificate issued?** This is the name of the company that will use this certificate.
2. **Who issued the certificate?** This is the certificate authority that issued this certificate.
3. **Validity period.** An expired certificate should not be trusted.

Your web browser handles this automatically, ensuring that you are talking with the correct server and that your communication is secure.

## Modern Certificate Ecosystem

The certificate landscape has evolved significantly:

**Let's Encrypt** launched in 2015 and provides free, automated TLS certificates. This removed the cost barrier that previously prevented many websites from using HTTPS. As a result, HTTPS adoption has grown from under 50% of web traffic in 2015 to over 95% today.

**Certificate Transparency (CT)** requires CAs to log all issued certificates to public, auditable logs. Browsers can check these logs and reject certificates that are not properly logged. This makes it much harder for attackers to obtain fraudulent certificates without detection.

**Short-lived certificates** are becoming more common. Let's Encrypt certificates are valid for only 90 days, encouraging automation and reducing the window of exposure if a certificate is compromised. Some organisations use certificates valid for just hours or days.

**Automated Certificate Management Environment (ACME)** is the protocol used by Let's Encrypt and other CAs to automate certificate issuance and renewal. Tools like Certbot make it straightforward to obtain and renew certificates automatically.

## Testing TLS Configurations

As a security professional, you may need to assess TLS configurations. Useful tools include:

- **testssl.sh**: A command-line tool that checks a server's TLS configuration for supported protocols, cipher suites, and common vulnerabilities. It is the best choice for detailed assessments, especially against internal systems that are not publicly accessible.
- **sslyze**: A Python tool for analysing SSL/TLS configurations, useful for automation and integration into CI/CD pipelines.
- **SSL Labs (ssllabs.com)**: A web-based service that provides detailed analysis of public-facing HTTPS servers. It is the quickest option for a one-off assessment of a public website.
- **nmap ssl-enum-ciphers**: An Nmap script that enumerates supported cipher suites as part of a broader port scan.