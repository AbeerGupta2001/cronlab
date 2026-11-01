### Mask Attacks
- It is a structured Brute-force attack in which you define the **pattern** of the password rather than just the character set.

| Placeholder | Character Set           |
| ----------- | ----------------------- |
| ?l          | Lowercase letters (a-z) |
| ?u          | Uppercase letters (A-Z) |
| ?d          | Digits (0-9)            |
| ?s          | Special characters      |
| ?a          | All printable ASCII     |
A mask for a password like `Summer2026!` would be `?u?l?l?l?l?l?d?d?d?d?s`. This generates far fewer candidates than a full brute force of the same length.

For hash identification we use `hashid` or `hashcat --identify`