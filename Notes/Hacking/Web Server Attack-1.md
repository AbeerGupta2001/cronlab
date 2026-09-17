## The Server Response Header

The most direct fingerprinting signal is the `Server` header in an HTTP response. When you make any request to a web server, the server includes this header in its reply. Different server software formats it differently, and those formats are consistent enough to use as positive identifiers.

Run `curl` with the `-I` flag to request only the headers, not the response body:

```bash
# -s suppresses the progress bar
# -I sends a HEAD request, returning only response headers
curl -sI http://MACHINE_IP:80
```

You should see output similar to the following for Apache running on port 80:

```shell-session
ubuntu@tryhackme-2404:~$ curl -sI http://MACHINE_IP:80
HTTP/1.1 200 OK
Date: Wed, 08 Apr 2026 13:59:00 GMT
Server: Apache/2.4.58 (Ubuntu)
Last-Modified: Fri, 03 Apr 2026 18:12:44 GMT
ETag: "29af-64e9243796aa2"
Accept-Ranges: bytes
Content-Length: 10671
Vary: Accept-Encoding
Content-Type: text/html
```

## The X-Powered-By Header

Some frameworks add an `X-Powered-By` header that reveals the application layer behind the server. Express sets this by default:

```text
X-Powered-By: Express
```

This header is separate from `Server`, and for Express, it is the primary fingerprint. Unlike Apache or Nginx, which announce themselves in the `Server` header, Express relies on `X-Powered-By` as its identifier. Check for it on any port where `Server` is missing or generic.

## Default Error Pages

The `-sI` flag sends a `HEAD` request, which returns headers only and no body. To see default error pages, you need a `GET` request switch to `-s` without the `-I`:

```bash
# HEAD request: headers only, no body
curl -sI http://MACHINE_IP:PORT/

# GET request: full response including body
curl -s http://MACHINE_IP:PORT/nonexistent-page-xyz
```

When you request a path that does not exist, most servers return a default 404 error page. Each server's default page has a distinctive look. Python is a plain text response. Nginx includes its version in the HTML footer. Apache includes its name in the page body. These differences let you fingerprint a server even when the `Server` header has been suppressed. Check this across all four ports, and you will see how differently each server handles the same situation.