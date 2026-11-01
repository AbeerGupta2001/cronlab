- **Crunch**: It takes a minimum and maximum length, and an optional character set, and outputs every possible strings that meets those criteria. For example `crunch 6 6 0123456789abcdef -o 6chars.txt`
- **CeWL:** It is a Ruby tool that spiders a URL and extracts unique words from the target. It can also harvest email address with the `-e` flag. For example

```bash
user@tryhackme:~$ cewl -d 2 -m 3 --lowercase --with-numbers -e --email_file emails.txt -w cewl_words.txt http://tryfinanceme.local

CeWL 6.2.1 (More Fixes) Robin Wood (robin@digi.ninja) (https://digi.ninja/)
```

Flags explained:

- `-d 2`: Spider two levels deep.
- `-m 3`: Only include words with three or more characters.
- `--lowercase`: Convert all extracted words to lowercase.
- `--with-numbers`: Include words that contain numbers.
- `-e`: Enable email extraction.
- `--email_file emails.txt`: Save found emails to `emails.txt`.
- `-w cewl_words.txt`: Save extracted words to `cewl_words.txt`.
- `http://tryfinanceme.local`: Target URL for crawling.

**Downloading Documents and Extracting Strings**
- `bash wget -r -A pdf http://tryfinanceme.local/docs/`
- `bashfor f in $(find tryfinanceme.local/docs -name '*.pdf'); do strings -n 5 "$f" | grep -vP '^[/<>%0-9\\]|^(stream|endstream|endobj|xref|trailer|startxref)$' >> raw_words.txt; done`

```bash
user@tryhackme:~$ grep -RhiaoP '[A-Za-z0-9._%+-]+@tryfinanceme\.com' tryfinanceme.local/docs > emails_docs.txt
user@tryhackme:~$ sort -u emails_docs.txt > emails_docs.unique.txt
user@tryhackme:~$ grep -Po '^[^@]+' emails_docs.unique.txt > users_from_emails.txt

user@tryhackme:~$ curl -s http://social.tryfinanceme.local/ | grep -Po '(?<=<h3 class="profile-name">)[^<]+' > names.txt

user@tryhackme:~$ awk '{print tolower($1)"."tolower($2)}' names.txt > users_first.last.txt
user@tryhackme:~$ awk '{print tolower(substr($1,1,1))tolower($2)}' names.txt > users_flast.txt
user@tryhackme:~$ awk '{print tolower($1)tolower(substr($2,1,1))}' names.txt > users_firstl.txt

        
```

**Creating and Cleaning Wordlists**

```
user@tryhackme:~$ cat cewl_words.txt raw_words.txt | sort -u > words_raw.txt

user@tryhackme:~$ cat words_raw.txt | tr '[:upper:]' '[:lower:]' | tr -d '\r' | grep -P '^[a-z0-9][a-z0-9._-]{4,}$' | sort -u > words_clean.txt

user@tryhackme:~$ cat users_first.last.txt users_flast.txt users_firstl.txt users_from_emails.txt | sort -u > users.txt
       
user@tryhackme:~$ crunch 11 11 -t Helios20%%! -o pass_helios.txt

         
```