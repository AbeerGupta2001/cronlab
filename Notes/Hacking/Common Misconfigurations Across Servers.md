
## Security Headers

**Security headers** are HTTP response headers that instruct the browser on how to handle the page content. They protect against a range of client-side attacks, including clickjacking, MIME sniffing, and cross-site scripting. None of the servers in this lab has been configured to send these headers, which is the default state for all four server types.

Here are the most common ones and what each one does:

|Header|What It Protects Against|Example Value|
|---|---|---|
|`X-Frame-Options`|Clickjacking (prevents the page from being embedded in an iframe on another domain)|`DENY` or `SAMEORIGIN`|
|`X-Content-Type-Options`|MIME sniffing (prevents the browser from guessing content types)|`nosniff`|
|`Content-Security-Policy`|Restricts where scripts, stylesheets, and other resources can load from|`default-src 'self'`|
|`Referrer-Policy`|Controls what is sent in the `Referer` header when navigating to another page|`no-referrer` or `strict-origin`|
|`Strict-Transport-Security`|Forces HTTPS for subsequent requests (only meaningful on HTTPS servers)|`max-age=31536000`|

![](https://cdn-images.tryhackme.com/room-icons/68d2c1e7ab94268f6271de1d-1771765213861)

**Note:** `X-Frame-Options` is technically superseded by `Content-Security-Policy: frame-ancestors`, which provides finer-grained control. Hardened modern sites may handle clickjacking protection through CSP alone and intentionally omit `X-Frame-Options`. When writing findings, check for both.

![](https://cdn-images.tryhackme.com/room-icons/68d2c1e7ab94268f6271de1d-1771765213861)

**Note:** `Strict-Transport-Security` only applies over HTTPS connections. Since this lab uses HTTP, its absence is expected and is noted here for completeness rather than as an active misconfiguration.

## Automated Scanning with Nikto

`Nikto` is a web server scanner that checks for known misconfigurations, outdated software, exposed admin interfaces, and missing security headers. It is not subtle; it generates a lot of traffic and is easy to detect. That makes it appropriate for authorised testing, not stealth. Run it against the Apache server:

```shell-session
root@ip-10-81-64-63:~# nikto -h http://MACHINE_IP:80 -nointeractive
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          MACHINE_IP
+ Target Hostname:    MACHINE_IP
+ Target Port:        80
+ Start Time:         2026-04-11 09:16:09 (GMT1)
---------------------------------------------------------------------------
+ Server: Apache/2.4.58 (Ubuntu)
+ Server leaks inodes via ETags, header found with file /, fields: 0x29af 0x64e9243796aa2 
+ The anti-clickjacking X-Frame-Options header is not present.
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Allowed HTTP Methods: HEAD, GET, POST, OPTIONS 
+ OSVDB-561: /server-status: This reveals Apache information. Comment out appropriate line in httpd.conf or restrict access to allowed hosts.
+ OSVDB-3268: /files/: Directory indexing found.
+ OSVDB-3092: /files/: This might be interesting...
+ 6544 items checked: 0 error(s) and 6 item(s) reported on remote host
+ End Time:           2026-04-11 09:16:18 (GMT1) (9 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```


The `-nointeractive` flag suppresses prompts so the scan runs without waiting for input. Nikto will check hundreds of common paths and patterns. Look for findings that start with `+` in the output.

On a misconfigured Apache server like this one, expect Nikto to flag the exposed `/server-status` page, the `backup.bak` file in the document root, the directory listing on `/files/`, and multiple missing security header warnings.

![](https://cdn-images.tryhackme.com/room-icons/68d2c1e7ab94268f6271de1d-1771765215788)

**Tip:** Nikto's output can be verbose. The `-Tuning` option lets you restrict which checks it runs. For a quick scan, `nikto -h TARGET -Tuning 123` covers the most common findings without running the full signature set. Note that tuning codes are concatenated, not comma-separated.

## Patterns That Apply Everywhere

Looking back at all four servers, the same categories of misconfiguration appear repeatedly:

| Misconfiguration                 | Apache                             | Python HTTP                       | Node.js                         | Nginx                                   |
| -------------------------------- | ---------------------------------- | --------------------------------- | ------------------------------- | --------------------------------------- |
| Version disclosure in headers    | Yes                                | Yes                               | Partial                         | Yes                                     |
| Directory listing                | `/files/`                          | Root path                         | N/A                             | `/files/`                               |
| Exposed status or debug endpoint | `/server-status`                   | N/A                               | `/api/debug/env`, `/api/routes` | `/nginx_status`                         |
| Sensitive files accessible       | `backup.bak`, `internal-notes.txt` | `.env`, `notes.txt`, `backup.zip` | `config.js`                     | `server-config.txt`, `deploy-notes.txt` |
| Missing security headers         | All                                | All                               | All                             | All                                     |
