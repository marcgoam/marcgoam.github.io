---
title: "Content Discovery"
---

# Web Content Enumeration

## Stack fingerprinting

### WhatWeb

`whatweb` fingerprints the HTTP stack: web server, framework, CMS, JS libs, analytics IDs, even JIRA versions. Run it first on any URL — it takes a second and often tells you exactly what to dig into next.

```bash
whatweb http://IP
```

### wafw00f

Before any fuzzing, check whether a WAF sits in front of the target — it'll shape which tool and rate you pick and explains weird 403s that would otherwise look like bugs in your scan.

```bash
wafw00f <domain>
```

---

## Directory and file enumeration

Content discovery is the bulk of web enum. Three tools do the same job with different ergonomics — pick whichever one fits the target. `gobuster` for simple structured output, `wfuzz` for complex filters, `ffuf` for speed and flexibility.

### gobuster — directories

Base directory brute with extension support. `-b` hides noisy status codes, `-k` skips TLS verification on self-signed certs.

```bash
gobuster dir -u <url> -w <wordlist.txt> -x <file_extensions>
```

```bash
gobuster dir -u <url> -w <wordlist.txt> -x <file_extensions> -b 403,404
```

HTTPS with cert issues:

```bash
gobuster dir -u <url> -w <wordlist.txt> -x <file_extensions> -b 403,404
```

```bash
gobuster dir -b 403,404 -u <url> --wordlist <wordlist.txt> -t 100 -k
```

**API pattern brute-force** — `gobuster -p` takes a patterns file where `{GOBUSTER}` expands to each word in the wordlist, good for versioned APIs. Example pattern file:

```
{GOBUSTER}/v1
{GOBUSTER}/v2
```

Run it:

```bash
gobuster dir -u <url> -w /usr/share/wordlists/dirb/big.txt -p pattern
```

### wfuzz

`wfuzz` shines when the target has wonky response sizes and you need to filter by multiple criteria at once.

```bash
wfuzz -c --hc 404 -t 200 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt https://example.com/FUZZ
```

The filter flags:

- `--hs/ss "regex"` — hide/show by regex in response
- `--hc/sc CODE` — hide/show by response code
- `--hl/sl NUM` — hide/show by number of lines
- `--hw/sw NUM` — hide/show by number of words
- `--hh/sh NUM` — hide/show by number of chars

### ffuf

`ffuf` is the fastest and most flexible of the three. `FUZZ` is a placeholder you can place anywhere in the URL.

```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ
```

Filters mirror the wfuzz list:

- `--fr "regex"` — hide/show by regex
- `--fc/sc CODE` — hide/show by code
- `--fl/sl NUM` — hide/show by lines
- `--fw/sw NUM` — hide/show by words
- `--fs/sh NUM` — hide/show by chars

**Extension fuzzing** — brute-force which extension a known page uses (e.g. `index.php` vs `index.html`):

```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/blog/indexFUZZ
```

**Page fuzzing** — find pages under a known directory:

```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php
```

**Recursive scanning** — let ffuf dive into directories it finds:

```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v
```

---

## DNS subdomain enumeration

Subdomain brute uses DNS (not HTTP), so make sure the target is DNS-resolvable. `--wildcard` tells gobuster to ignore catch-all wildcards.

### gobuster

```bash
gobuster dns -d <domain> -w <wordlist.txt> -i --wildcard
```

### wfuzz

```bash
wfuzz -c --hc=400,404,404 -t 20 -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-
top1million-20000.txt -H "Host: FUZZ.example.com" -u http://example.com  
```

### ffuf

```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u https://FUZZ.inlanefreight.com/
```

---

## VHost enumeration

VHost enum is different from DNS subdomain enum: you hit a single known IP and vary the `Host:` header to discover sites served from the same server. Useful when subdomains aren't DNS-registered but are still served.

```bash
gobuster vhost -u <RHOST> -t 50 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt
gobuster vhost -u <RHOST> -t 50 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```

```bash
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb'
```

---

## Request / parameter fuzzing

When you know a specific endpoint exists, fuzzing the parameter name or value tells you what the app expects. `ffuf -request` replays a raw request file (from Burp), which is the cleanest way to fuzz complex multi-header requests like SSRF or SQLi-suspect endpoints.

```bash
ffuf -request ssrf.req -request-proto http -w <(seq 1 65535) -debug-log /dev/stdout

ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx

ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```

---

## Nikto

`nikto` is the classic "quick vuln scan" — slow, noisy, and prone to false positives, but occasionally spots misconfigs the fuzzers miss (CGI, backup files, default creds). `-Tuning b` restricts to "software identification" — fingerprint-only, no active checks.

```bash
nikto -h inlanefreight.com -Tuning b
```

---

## CMS-specific enumeration

### WordPress — wpscan

The reference WordPress scanner. API token (free to register) unlocks the full vulnerability DB; without it you still get the basic user/theme/plugin enum.

Automatic scan:

```bash
wpscan --url http://example.com --api-token "token"
```

User enumeration:

```bash
wpscan --url https://example.com --enumerate u
```

Brute-force against discovered user:

```bash
wpscan --url example.com -U user -P /usr/share/wordlists/rockyou.txt
```

Additional variants — themes/plugins/users together, aggressive plugin detection, TLS-bypass for broken certs:

```bash
wpscan --url https://<RHOST> --enumerate u,t,p
wpscan --url https://<RHOST> --plugins-detection aggressive
wpscan --url https://<RHOST> --disable-tls-checks
wpscan --url https://<RHOST> --disable-tls-checks --enumerate u,t,p
wpscan --url http://<RHOST> -U <USERNAME> -P passwords.txt -t 50
```

### WordPress — manual tricks

Version identification without wpscan — check the `<meta>` generator tag, `readme.html` / `license.txt`, HTTP response headers, or the REST API at `/wp-json/v2/user`. Quick greps:

```bash
curl -s <url> | grep WordPress
```

Enumerate plugins:

```bash
curl -s <url> | grep plugins
```

Enumerate themes:

```bash
curl -s <url> | grep themes
```

REST API user enum:

```bash
curl https://xxx/wp-json/wp/v2/users
```

**Dictionaries**:

- `/usr/share/wordlists/SecLists/Discovery/Contenido Web/CMS/wordpress.fuzz.txt`
- `/usr/share/wordlists/SecLists/Discovery/Web-Content/CMS/wp-themes.fuzz.txt`

**Login / auth paths to check**:

- `/wp-login.php` (often renamed to `login.php`)
- `/wp-admin/login.php`
- `/wp-admin/wp-login.php`
- `xmlrpc.php`

**Important directories**:

- `/wp-content` — plugins and themes
- `/wp-content/uploads/` — uploaded files
- `wp-config.php` — DB credentials

### Joomla — droopescan

`droopescan` is the multi-CMS scanner — covers Joomla, Drupal, SilverStripe, and more.

```bash
droopescan scan joomla --url http://joomla-site.local/
```

Versions **4.0.0 to 4.2.7** are vulnerable to unauthenticated information disclosure (CVE-2023-23752), which dumps credentials and other data.

- Users: `http://<host>/api/v1/users?public=true`
- Config File: `http://<host>/api/index.php/v1/config/application?public=true`

### Drupal — droopescan

```bash
droopescan scan drupal --url https://example.com
```

---

## Exposed .git / source

When the target's `.git` directory leaks, `GitTools` pulls it down and reconstructs the working tree. Old commits frequently contain credentials, comments, and removed files worth looting.

```bash
./gitdumper.sh http://<RHOST>/.git/ /PATH/TO/FOLDER
./extractor.sh /PATH/TO/FOLDER/ /PATH/TO/FOLDER/
```

Walk the repo afterwards — especially `git log` for credential commits and `git show` for specific commits.

```bash
git log
git show <hash>
```

---

## Building target-specific wordlists — cewl

`cewl` crawls a site and returns every word it finds. The resulting wordlist is gold for password spraying and for targeted brute-forcing of forms and logins on the same target.

```bash
# -m -> Minimum word length. Only words with at least 5 character
cewl http://url -m 5 -w wordlist.txt
```

---

## Crawlers — ReconSpider

`ReconSpider` is a Scrapy-based crawler that enumerates URLs/endpoints for a given domain. Setup then run:

```bash
pip3 install scrapy
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
unzip ReconSpider.zip 
```

```bash
python3 ReconSpider.py http://inlanefreight.com
```



