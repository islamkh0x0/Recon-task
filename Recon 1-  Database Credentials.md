## 1. Preparation:

setting up a structured environment to manage the data you’ll collect.
Directory Structure
```bash
mkdir -p example.com ~/recon/targets/example.com/subdomains/  
mkdir -p example.com ~/recon/targets/example.com/endpoints/  
mkdir -p example.com ~/recon/targets/example.com/aws/  
mkdir -p example.com ~/recon/targets/example.com/dns/
```
`-p`: Ensures parent directories are created if they don’t exist, avoiding errors.

---
## 2. Subdomain Enumeration
-  Subfinder: 
   Passive: Queries open sources like VirusTotal, Censys, and Shodan without sending requests to the target.
   Active: Performs DNS queries or light brute-forcing to find additional Subdomains
   ```bash
   subfinder -d example.com -o ~/recon/targets/example.com/subdomains/subfinder.txt
   ```
   `-o`: Outputs results to subfinder.txt.
   
-  Assetfinder: use Certificate Transparency (CT) Logs to find Subdomains.
```bash
assetfinder --subs-only example.com >> ~/recon/targets/example.com/subdomains/assetfinder.txt
   ```

-  Alterx:
   Dynamic Enumeration
	 `alterx -enrich`: Adds dynamic variations (e.g., dev, test) to the domain.
	`dnsx`: Resolves the generated Subdomains via DNS.
	
```bash
   echo example.com | alterx -enrich | dnsx > ~/recon/targets/example.com/subdomains/alterx-dynamic.txt
```

 Permutation
```bash
  echo example.com | alterx -pp 'word=subdomains-top1million-50000.txt' | dnsx > ~/recon/targets/example.com/subdomains/alterx-permutation.txt
```
Wordlists are available at SecLists on GitHub.

-  Asnmap:
   target’s  (ASN) to discover related IPs and Subdomains.
   
```bash
asnmap -d example.com | dnsx -silent -resp-only -ptr > ~/recon/targets/example.com/subdomains/dnsx.txt
```

-  Ffuf
   performs Virtual Host (VHost) enumeration to find Subdomains not in DNS.
   
   ```bash
   cat subdomains-top1million-50000.txt | ffuf -w -:FUZZ -u http://example.com/ -H 'Host: FUZZ.example.com' -ac
```

---

## 3. Filtering Subdomains
After enumeration, combine and filter Subdomains to create a clean, actionable list.
#### Merge Results
```bash
cat ~/recon/targets/example.com/subdomains/*.txt | anew ~/recon/targets/example.com/subdomains/subdomains.txt
```

#### Filter Live Subdomains
```bash
cat ~/recon/targets/example.com/subdomains/subdomains.txt | httpx -o ~/recon/targets/exampl.com/subdomains/httpx.txt
```


---

after we filter live subdomains we will start a information gathering using httpx to gather infomation about the domains using **wappalyzer mapping** _techniques_ to identify technologies that are used to websites.

```bash
cat ~/recon/targets/example.com/subdomains/httpx.txt | httpx -cname -status-code -td -o ~/recon/targets/example.com/subdomains/tech.txt
```

**-cname**: Shows CNAME records (e.g., api.example.com → cdn.example.com).
**-td**: Detects technologies (e.g., Nginx, PHP).

---
## nuclei time
after we gather information about tech we will start using nuclei
> Nuclei is used to send requests across targets based on a template, leading to zero false positives and providing fast scanning on a large number of hosts. Nuclei offers scanning for a variety of protocols, including TCP, DNS, HTTP, SSL, File, Whois, Websocket, Headless, Code etc. With powerful and flexible templating, Nuclei can be used to model all kinds of security checks.

```bash
cat ~/recon/targets/example.com/subdomains/httpx.txt | nuclei -config ~/nuclei-templates/config/custom.yml
```
#### config file 
```yml
# nuclei -config ~/nuclei-templates/config/custom.yml -list target_list_to_scan.txt

severity:
  - critical
  - high
  - medium
  - low

type:
  - http
  - tcp
  - javascript

include-tags:
  - generic
  - config
  - misconfig
  - exposures
  - exposure
  - disclosure
  - file
  - logs
  - traversal
  - xss
  - lfi
  - crlf
  - cache
  - takeovers
  - wordpress

exclude-tags:
  - tech
  - dos
  - fuzz
  - creds-stuffing
  - token-spray
  - osint
  - headers # exlude finding missing HTTP security headers
```


[[Full Details- recon 1]]
