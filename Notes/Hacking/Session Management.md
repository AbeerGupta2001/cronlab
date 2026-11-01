
### Session Management Lifecycle

- **Session Creation**

	You might think this first step in the lifecycle occurs only after you provide your credentials, such as a username and password. However, on many web applications, the initial session is already created when you visit the application. This is because some applications want to track your actions even before authentication. However, our main focus for this room will be on authenticated sessions. Once you provide your username and password, you receive a session value that is then sent with each new request. How these session values are generated, used, and stored is crucial in securing session creation.  

- **Session Tracking**

	Once you receive your session value, this is submitted with each new request. This allows the web application to track your actions even though the HTTP protocol is stateless in nature. With each request made, the web application can recover the session value from the request and perform a server-side lookup to understand who the session belongs to and what permissions they have. In the event that there are issues in the session tracking process, it may allow a threat actor to hijack a session or impersonate one.  

- **Session Expiry**

	Because the HTTP protocol is stateless, it may happen that a user of the web application all of a sudden stops using it. For example, you might close the tab or your entire browser. Since the protocol is stateless, the web application has no method to know that this action has occurred. This is where session expiry comes into play. Your session value itself should have a lifetime attached to it. If the lifetime expires and you submit an old session value to the web application, it should be denied as the session should have been expired. Instead, you should be redirected to the login page to authenticate again and start the session management lifecycle all over again!  

- **Session Termination**

	However, in some cases, the user might forcibly perform a logout action. In the event that this occurs, the web application should terminate the user's session. While this is similar to session expiry, it is unique in the sense that even if the session's lifetime is still valid, the session itself should be terminated. Issues in this termination process could allow a threat actor to gain persistent access to an account.

### Authentication vs Authorisation

To understand the common vulnerabilities in session management, we first need to examine authentication and authorisation. While they sound the same and are often confused, each plays a critical and unique role in session management.

#### Identification

Identification is the process of verifying who the user is. This starts with the user claiming to be a specific identity. In most web applications, this is performed by submitting your username. You are claiming that you are the person associated with the specific username. Some applications use uniquely created usernames, whereas others will take your email address as the username.  

#### Authentication

Authentication is the process of ensuring that the user is who they say they are. Where in identification, you provide a username, for authentication, you provide proof that you are who you say you are. For example, you can supply the password associated with the claimed username. The web application can confirm this information if it is valid; this is the point where session creation would kick in.  

#### Authorisation

Authorisation is the process of ensuring that the specific user has the rights required to perform the action requested. For example, while all users may view data, only a select few may modify it. In the session management lifecycle, session tracking plays a critical role in authorisation.  

#### Accountability

Accountability is the process of creating a record of the actions performed by users. We should track the user's session and log all actions performed using the specific session. This information plays a critical role in the event of a security incident to piece together what has happened.  

#### IAAA and Session Management

Now that you understand the differences between authentication and authorisation let's bring this back to session management. Authentication plays a role in how sessions are created. Authorisation becomes important to verify that the user associated with a specific session has the permission to perform the action they are requesting. Accountability is crucial for us to piece together what actually occurred in an incident, which means it is important that requests are logged and that the session associated with each request is also logged.


### Cookies vs Tokens

#### Cookie-Based Session Management
- Also called the old school way of managing session
- The Set-Cookie header value will be sent in the response of creating a session for tracking the user.
- Several attributes can also be added to this header.
	- **Secure** - Indicates to the browser that the cookie may only be transmitted over a verified HTTPS channels. If there are certificate error or HTTP is used, the cookie value will not be transmitted
	- **HTTPOnly** - Indicates to the browser that the cookie value may not be read by client-side JavaScirpt.
	- **Expire** - Indicates to the browser when a cookie value will no longer be valid and should be removed.
	- **SameSIte** - Indicates to the browser whether the cookie may be transmitted in cross-site requests to help protect against CSRF attacks.

#### Token-Based Session Management
- Relatively new concept.
- uses client side code the process.
- After authentication, the web application provides a token within the request body.
- Token is stored in the browser localstorage.
- When a new request is made, Javascript fetch the token from localstorage and attach it to the header of the request.
- Common types of tokens is JSON Web Tokens (JWT), which is passed to the `Authorization: Bearer` header

| **Cookie-Session Management**                                                                                                                         | **Token-Based Session Management**                                                                                                                                        |
| ----------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Cookie is automatically sent by the browser with each request                                                                                         | Token has to be submitted as a header with each request using client-side JavaScript                                                                                      |
| Cookie attributes can be used to enhance the browser's protection of the cookie                                                                       | Tokens do not have automatic security protections enforced and should, therefore, be safeguarded against disclosures                                                      |
| Cookies can be vulnerable to conventional client-side attacks such as CSRF, where the browser is tricked into making a request on behalf of the user. | As the token is not automatically added to any request and cannot be read from LocalStorage by other domains, conventional client-side attacks such as CSRF are blocked.  |
| As cookies are locked to a specific domain, it can be difficult to use them securely in decentralised web applications.                               | Tokens work well in decentralised web applications, as they are managed through JavaScript and can often contain all the information required to verify the token itself. |

### Securing the Session Lifecycle

#### Session Creation
- It is where the most vulnerabilities can creep in.
- **Weak Session Value** 
	- Less common in modern times as frameworks are consistently used.
	- If a custom session creation mechanism has been implemented, there is a good chance that the session values may be guessable.
	- For example: Simply using base64 encoded username as session value.
- **Controllable Session Values** 
	- In certain tokens, such as JWTs, all the relevant information to both create and verify the JWT's validity is provided. If security measures are not enforced, such as verifying the token's signature or ensuring that the signature itself was created securely, a threat actor would be able to generate their own token.
- **Session Fixation**
	- A web application that gives session before authentication is vulnerable to session fixation
	- If our session value is not adequately rotated once we authenticate, a suitably positioned threat actor could record it and wait for our authentication to gain access to our session.

- **Insecure Session Transmission**
	- It common for the authentication server and application server to be distinct.
	- Like Single Sign-On (SS0) solutions.
	- For this to work session material should be transferred from the authentication server to application server via our browser.
	- Problem occurs during the transmission that would expose our session information to the threat actor.
	- The most common is an insecure redirect where the threat actor can control the URL where we will redirected to post-authentication.
	
- **Session Tracking** 
	- **Authentication Bypass** 
		- This happens when there aren't sufficient checks being performed on whether a user is allowed to perform the action they requested.
		- Two types:
			- Vertical bypass - We can perform an action that is reserved for more privileged user
			- Horizontal bypass - We can perform actions we are allowed but on a dataset that we should not be allowed to perform the action on.
		- Vertical bypass is easy to defend against since function decorator and path based access control configuration are used
		- However horizontal bypass, user actions are allowed to perform but the issue is they are performing on others data
	
	- **Insufficient Logging** 
		- Key issue not having sufficient information about the incident.
		- Logging can be crucial to understanding what went wrong.
		- It can leave gaps in the investigation that cannot be filled.
		- Logs should cover both accepted and rejected actions.

- **Session Expiry** 
	- Single vulnerability which is when the expiry time for sessions are excessive.

- **Session Termination**
	- Key issue is when sessions are not properly terminated server-side when the logout action is performed. 
	- This can be quite a issue for tokens where the lifetime of the token is embedded in the token itself.
	- Upon successful password reset, it is recommended that all sessions are terminated to allow a user to regain full control of their account.
	