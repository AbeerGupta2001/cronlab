Four Techniques:

- **Username Enumeration**
	- It is used to produce a list of accounts registered on the target application.
	- It is done by submitting usernames to a form that responds differently for registered and unregistered users, such as a signup or password reset page.
	- Further this list is used in credential attacks.
- **Credential Brute Force
	- Uses lists of common passwords and usernames to brute force an account and gain access to it.
	- If rate limiting is not applied, the attack is inexpensive to run and frequently produces a working credential in seconds
- **Logic Flaws** 
	- When a intended flow of an authentication-related workflow can be redirected by valid-looking input. 
	- Password reset and account recovery workflows are frequent source of such flaws because they commonly split input across multiple HTTP parameters, cookies, and session variables.
- **Cookie manipulation**
	- Target the session state that the server returns to the client after a successful login.
	- Cookies that are not cryptographically signed can be edited by the client to change the authentication decision the server make on subsequent requests.
	- Common format are plain text, cookies containing hash of underlying value and cookie containing a reversibly encoded payload.


	