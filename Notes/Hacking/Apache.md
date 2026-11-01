
**Apache** is the most widely deployed web server in the world, which means it appears in nearly every engagement that touches web infrastructure. Its default Ubuntu configuration leaves several things enabled that testers regularly find and report. The three most common are directory listing on specific paths, the `server-status` module, and backup files sitting in the document root.

## Directory Listing

Apache's `Options +Indexes` directive tells the server to display a file listing when a directory does not have an `index.html` or similar default file. This is sometimes enabled intentionally for internal file share directories and accidentally left on paths that contain sensitive data.

Browse to the `http://10.48.159.160/files/`

You should see an HTML page with a title of "`Index of /files`" listing the files available in that directory. Apache's directory listing includes filenames, file sizes, and last-modified dates, which gives you a clear picture of what is there before you decide what to retrieve.

When you find a directory listing, read every file in it. Developers sometimes leave CSV exports, internal notes, or backup files in directories they intended only for casual internal use. Each one is worth checking.

## The mod_status Page

Apache includes a built-in status page powered by the `mod_status` module. When correctly configured, it is accessible only from localhost. When misconfigured with `Require all granted`, it is accessible from any IP. You can access it using `http://10.48.159.160:80/server-status`.

**Info:** `mod_status` is enabled by default on Ubuntu's Apache package and ships with a `Require local` restriction in `conf-available/security.conf`. However, a `Require all granted` directive anywhere in a virtual host configuration silently overrides that restriction, exposing `/server-status` to all IPs without touching the module config. Always check `/server-status` even on servers that appear to be using default settings.

## Finding Unlinked Files with Gobuster

Not everything interesting is linked from a page or visible in a directory listing. Backup files, old configuration copies, and forgotten test files often sit in the document root with no links pointing to them. Tools like `Gobuster` discover these by guessing paths against a wordlist.When Gobuster finds a file with a `.bak` extension, that is almost always worth retrieving. Backup files frequently contain configuration snippets, credentials, or copies of source code.

Also, pay attention to `.htpasswd` files if they appear in the output. Apache uses `.htpasswd` to store usernames and hashed passwords for HTTP Basic Authentication. Finding an accessible `.htpasswd` file gives you a credential hash that can be cracked offline, and confirms that some part of the site uses Basic Auth, which tells you what paths are worth trying authenticated access on.