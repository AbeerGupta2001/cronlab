- Phishing is one of the most powerful tools in a penetration tester's arsenal.
- It is a form of cyber attack that uses social engineering to trick people into revealing sensitive information or running malware on their devices.
- The primary channels for phishing attacks include emails, SMS(known as smishing), voice calls(vishing) and fake website designed to look legitimate.
- Types of Phishing
	- **Phishing**: It is the scam's broad, "cast a wide net" version. Attackers send the same believable message to many people at once, often using common themes like account alerts or invoices.
	- **Spear Phishing:** It is targeted attack tailored to a specific person. The goal is usually to get the target to click a link, open a file, run a task, or submit credentials so the attacker can move deeper into the organisation's network.
	- **Whaling:** It is spear phishing that targets senior decision-makers and executives, like CEOs and CFOs. Both spear phishing and whaling are target and customized attack; the distinction is who is targeted and what leverage is expected. 

### Psychology of Phishing
- **Social engineering principles**:
	- **Scarcity:** It makes something feel rare, which pushes people to act before they think. Psychological, **FOMO** (fear of missing out) and loss aversion kick in: we dislike losing a chance more than we like gaining a benefit. We can use terms like **limited seats**, **last chance**, or **ends today**. For example "Only three TryPhones up for grabs."
	- **Urgency**: Urgency adds a countdown so the brain priorities speed over scrutiny. Time pressure narrow attention and reduces deliberate checking. For examples "Your account will be suspended in 12 hours unless you verify your identity through this portal".
	- **Authority:** People are more likely to follow instruction if they think they come from leaders, experts or official departments. For example "From IT Administrator. Action required on your SSO settings."
	- **Fear:** It uses threat and alarm to trigger a protective reaction, pushing people to fix the problem immediately. Wording often includes "security alert", "breach" or "unauthorized access". For example "We detected suspicious logins on your account. Secure it immediately here."
	- **Curiosity:** It hooks attention by promising interesting information. The brain wants to close information gaps, which can outweigh caution when the tease feels relevant or exclusive. For example "Confidential: Q3 roadmap highlights."
	- **Trust:** It piggybacks on familiar brands, colleagues or communication styles so the message feels safe by default. For example "Microsoft 365: New security notice available in your portal."
- **Cognitive Biases:**
	- It is the tendency to make decisions based on feelings, assumptions or past experiences instead of facts. 
		- **Overconfidence bias:** Many people, especially cyber security practitioners, think they are too smart to fall for phishing scams
		- **Confirmation bias:** This happens when people accept information that fits their expectations. For example, if someone is waiting for an email from the bank, they might trust a phishing email that pretends to be from bank without verifying it.
		- **Authority bias:** This leads people to trust messages from those they see as authority figures without question.

### Phishing Techniques
- **URL and Domain Manipulation**:
	- **URL Masking:** Involves disguising a malicious URL behind a legitimate-looking hyperlink. For example an attacker might display https://tryhackme.com while redirecting users to http://phisher.thm
	- **Homograph Attacks:** Exploit visual similarities between domain name characters for example replacing "o" with "0" or using Cyrillic characters. 
	- **Typosquatting**: Involves registering domains similar to legitimate ones, relying on users making typing errors, for example, `tryhacme.com` instead of `tryhackme.com` 
- **Email Spoofing Fundamentals**:
	- It is a technique for impersonating a legitimate sender by modifying email headers. For example we can spoof the "From" field to display a trusted sender's email address like a manager or someone from HR.
	- If a domain is lacking security measures for authentication, an attacker can use a Python script to modify their email address. This is possible because SMTP does not have built-in functionality for authenticating email addresses.
	- Display name spoofing involves changing the sender's name in an email client while keeping the actual email address hidden.
	- ```html
		From: Support <support@tryaccounting.thm> 
		To: bob@tryaccounting.thm
		Subject: Urgent: Account Verification Required
		
		Dear Bob,
		
		As part of our security policy, we require all TryAccounting employees to change their passwords every 3 months. Please log in to our internal portal and update your password before Friday:
		http://tryaccounting-security.thm/account
		
		Thank you,
		TryAccounting Support Team
		```
	- Many organisations use security measures such as **SPF** (Sender Policy Framework), **DMARC** (Domain-based Message Authentication, Reporting and Conformance), and DKIM (DomainKeys Identified Mail), to help prevent such attacks. 

### Credential Harvesting:
- In a login cloning attack, the attacker replicates all visual elements of legitimate website, including logos, fonts and form fields and hosts them on a deceptive domain.
- The credentials are logged in a database or file. The user is redirected to the original website with error of failed to login.

### Payload Delivery Mechanisms
- A frequently used delivery method involves a Microsoft Word Document containing a marco. Upon opening the file, victim encounters a prompt to Enable Content in order to view the document, which serves as a built-in social engineering tactic. Once enabled, the VBA marco execute silently in the background.

### Tools of the Trade
- **GoPhish**: It a web-based framework that makes setting up phishing campaigns more straightforward. It allows us to store our SMTP server settings for sending emails and has a web-based tool for creating email templates using a simple WYSIWYG(What you see is what you get) editor.
- **EvilNginx:** It is a tool designed for advanced phishing campaigns that bypass multi-factor authentication (MFA). It acts as a reverse proxy between victims and legitimates sites, capturing credentials and session tokens in real time.
- **The Social Engineering Toolkit (SET):** It contains many tools. 

### Anatomy of Phishing Campaign:

![[Pasted image 20261025070840.png|475]]

|Metric|What it measures|Benchmark|Suggested Recommendation(s)|
|---|---|---|---|
|Open Rate|% of users who opened the email.|Industry varies; typical phishing open rates ~50–65%|Targeted refresher training|
|Click Rate|% of all users who clicked a link.|8–14% acceptable; >14% high risk|Focused security awareness training|
|Credential Entry Rate|% of all users who entered credentials after clicking.|<2% low risk; 2–5% moderate risk; >5% high risk|Phishing site identification training, MFA implementation|
|Attachment Detonation Rate|% of users who opened/executed an attachment.|No formal benchmark; >5–7% suggests risk|Educate on safe handling of attachments, Sandbox detonation|
|Reporting Rate (24h)|% of users who reported the email within 24h.|>40% strong; 30–40% average; <30% low|Reporting awareness campaign|
