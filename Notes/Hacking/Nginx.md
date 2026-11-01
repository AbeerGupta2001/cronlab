
After Apache and Node.js, the **Nginx** investigation follows the same template but with its own configuration vocabulary. The categories of misconfiguration version disclosure, directory listing, and exposed status pages appear again, but the specific directives and paths differ.

Nginx occupies a different space from Apache and Node.js. It is most commonly used as a reverse proxy, a load balancer, or a high-performance static file server. In production environments, Nginx often sits in front of an application server and handles the public-facing traffic. That positioning makes its configuration important. A misconfigured Nginx instance can expose internal structure, reveal operational data, or make files accessible that should not be.

![](https://cdn-images.tryhackme.com/room-icons/68d2c1e7ab94268f6271de1d-1771765216406)

**Info:** In the attached VM, Nginx runs on port 8080 rather than the standard port 80. That is because Apache is already using port 80 on this machine. In a real deployment, Nginx would typically run on port 80 or 443.

**Version Disclosure**

The pattern is the same as before: start with the `Server` header.

```shell-session
root@ip-10-81-64-63:~# curl -sI http://10.48.159.160:8080 | grep -i server
Server: nginx/1.24.0 (Ubuntu)
```

## Directory Listing with Autoindex

Nginx does not enable directory listing by default. When a developer wants to expose a directory listing, they add `autoindex` to a location block in the configuration:

```nginx
location /files/ {
    autoindex on;
    root /var/www/nginx/;
}
```

This is a documented Nginx configuration option and appears in legitimate file-sharing setups. The misconfiguration is using it on a path that contains sensitive files, or leaving it enabled on a production server without access controls.

Browse to the `http://10.48.159.160/files/` directory:

```shell-session
root@ip-10-81-64-63:~# curl -s http://10.48.159.160:8080/files/
<html>
<head><title>Index of /files/</title></head>
<body>
<h1>Index of /files/</h1><hr><pre><a href="../">../</a>
<a href="deploy-notes.txt">deploy-notes.txt</a>                                   03-Apr-2026 18:23                 148
<a href="old-backup.tar.gz">old-backup.tar.gz</a>                                  03-Apr-2026 18:23                 236
<a href="server-config.txt">server-config.txt</a>                                  03-Apr-2026 18:23                 135
</pre><hr></body>
</html>
```

Nginx's autoindex HTML format uses a simple table with filename, last-modified date, and file size. When you find this, read every file listed. Nginx directory listings often appear on paths that developers configured as shared storage and then populated with operational data.

## The nginx_status Endpoint

Nginx's `stub_status` module exposes real-time connection metrics at a configurable URL. The secure configuration restricts access to localhost only. The misconfigured version allows access from any IP:

```nginx
location /nginx_status {
    stub_status;
    allow all;  # Should be: allow 127.0.0.1; deny all;
}
```

Request it to see what it exposes:

```shell-session
root@ip-10-81-64-63:~# curl -s http://10.48.159.160:8080/nginx_status
Active connections: 1 
server accepts handled requests
 15 15 15 
Reading: 0 Writing: 1 Waiting: 0 
```

The output format is compact and uses unlabeled numbers on the second line, which can be disorienting the first time you see it:

The three numbers on the second line are in order: total accepted connections, total handled connections, and total requests since the server started. The third line breaks down currently active connections by state. While this data is not directly exploitable, it leaks operational information about server load and usage patterns. On a real engagement, an exposed `/nginx_status` endpoint is a finding because it confirms the internal monitoring setup and may indicate other monitoring endpoints are similarly exposed.

**Tip:** Nginx configuration files live in `/etc/nginx/` on Ubuntu. If you ever have shell access to a machine running Nginx, reading the site configuration in `/etc/nginx/sites-available/` will show you exactly what directories are exposed and what modules are enabled.