
# Subdomain Enumeration

**Subdomain enumeration** is the process of finding valid subdomains for a domain.
But why do we do this?

We do this to **expand our attack surface** in order to discover additional services, applications, and potential points of vulnerability that are not exposed on the main domain.

In real-world engagements, subdomains often host:

* Development or staging environments
* Administrative panels
* APIs
* Legacy applications

---

## Subdomain Enumeration Techniques

There are **three primary subdomain enumeration methods** that every pentester must understand:

* **OSINT (Open-Source Intelligence)**
* **Brute Force**
* **Virtual Host Enumeration**

Each method discovers subdomains in a different way, and **no single method is sufficient on its own**.

---

## OSINT-Based Subdomain Enumeration

### Certificate Transparency (CT Logs)

**Website:** `crt.sh`

When an SSL/TLS (Secure Sockets Layer / Transport Layer Security) certificate is created for a domain by a Certificate Authority (CA), the CA is required to log the certificate in **Certificate Transparency (CT) logs**.

These logs are publicly accessible and contain both **current and historical certificates** issued for a domain.

The purpose of CT logs is to detect malicious or accidentally issued certificates. However, we can leverage them to discover subdomains belonging to a domain.

Websites such as:

```
https://crt.sh
```

allow us to search these logs and extract subdomain names that were included in SSL certificates.

---

### Search Engine Enumeration

Search engines index trillions of web pages, making them an excellent OSINT source for discovering subdomains.

Using advanced search operators such as `site:` allows us to narrow results to a specific domain.

**Example:**

```
site:*.domain.com -site:www.domain.com
```

Explanation:

* `site:*.domain.com` → returns all indexed pages for the domain
* `-site:www.domain.com` → excludes the main domain

This query reveals **only subdomains** belonging to `domain.com`.

---

### Automating OSINT Enumeration

Manually querying search engines and CT logs is slow.
To automate OSINT-based enumeration, we use tools.

---

### Sublist3r (Outdated)

```
sublist3r -d acmeitsupport.thm
```

Sublist3r aggregates search engines and public sources to find subdomains.
However, it is **outdated** and should generally be replaced by more modern tools.

---

### assetfinder

The tool **assetfinder** aggregates data from multiple public sources, including:

* crt.sh
* certspotter
* ThreatCrowd
* VirusTotal
* Facebook Graph API
* Wayback Machine

**Example usage:**

```
assetfinder example.com
```

**Common options include:**

* `-subs-only` — show only subdomains
* `-o` — save output to a file
* `-resolve` — resolve subdomains to IPs
* `-live` — check for live hosts

assetfinder is **purely passive**, does **not brute force**, and returns **unvalidated results**.

---

### subfinder (Recommended)

**subfinder** is a modern replacement for Sublist3r and is considered one of the **best passive subdomain enumeration tools** available today.

---

## DNS Bruteforce Subdomain Enumeration

Bruteforce DNS enumeration involves attempting many possible subdomain names using a predefined wordlist.

This method sends DNS queries directly to name servers and is therefore **active and noisy**.

Because this process can involve thousands of requests, it must be automated.

---

### dnsrecon

```
dnsrecon -t brt -d acmeitsupport.thm
```

dnsrecon performs DNS brute forcing and can also test for zone transfers.

---

### amass (Active Mode)

amass is more comprehensive but considerably slower than subfinder.

It has two modes:

* **Passive mode** — uses OSINT and Certificate Transparency
* **Active mode** — performs DNS brute force and resolution

**Example passive usage:**

```
amass enum -passive -d target.com
```

Active mode switches to DNS brute forcing and should be used cautiously depending on engagement scope.

---

## Virtual Host Subdomain Enumeration

Some subdomains are **not publicly accessible via DNS**.

Examples include:

* Development environments
* Internal admin panels
* Applications defined only in `/etc/hosts`

Web servers can host multiple websites on the same IP address.
The server determines which website to serve based on the **Host header** in the HTTP request.

By fuzzing the Host header, we can discover virtual hosts that do not exist in DNS.

---

### ffuf (Virtual Host Discovery)

```
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/namelist.txt \
-H "Host: FUZZ.acmeitsupport.thm" \
-u http://10.x.x.x \
-fs {size}
```

In this command:

* `FUZZ` is replaced with subdomain candidates
* The response size filter removes the default response
* Filtering can also be done by status code or word count

This technique is **critical for modern web applications behind reverse proxies**.

---

## Validation and Resolution (Critical Step)

Subdomain enumeration results are often noisy and contain false positives.
Enumeration alone does **not** guarantee that a subdomain is valid or reachable.

---

### dnsx

`dnsx` is used to resolve discovered subdomains and verify which ones have valid DNS records.

---

### massdns

`massdns` is a high-performance DNS resolver designed for large wordlists and high-speed resolution.

---

### httpx

`httpx` is used to identify which resolved subdomains are **actually hosting web services**, and on which ports and protocols.

This validation phase ensures:

* Dead subdomains are removed
* Only live attack surface remains

---

## Summary — Tools Used

### DNS & Enumeration Tools

* dnsrecon
* amass
* assetfinder
* subfinder
* sublist3r *(outdated)*

### Virtual Host Enumeration

* ffuf

### Validation & Resolution

* dnsx
* massdns
* httpx

### Websites

* crt.sh


Just say which.
