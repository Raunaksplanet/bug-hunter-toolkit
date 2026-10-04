# Bug Hunter Toolkit

> End-to-end bug bounty workflow kit: subdomain recon → URL collection → vulnerability testing → reporting. Scripts + methodology + AI prompts + wordlists in one place.

[![Bash](https://img.shields.io/badge/scripts-bash-4EAA25?logo=gnubash&logoColor=white)](./01-Recon-Subdomains/)
[![Python](https://img.shields.io/badge/scripts-python-3776AB?logo=python&logoColor=white)](./03-Testing-Attacks/)
[![Burp Suite](https://img.shields.io/badge/tools-burp_suite-FF6633?logo=burpsuite&logoColor=white)](./06-Burp-Extensions/)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen)](./)
[![For](https://img.shields.io/badge/for-authorized_testing_only-red)](./)

All scripts are personally developed for **authorized security testing and bug bounty hunting only**. Not responsible for any harmful or illegal use. Use only on targets you have explicit permission to test.

---

## What is this?

A practical toolkit that covers the full bug bounty loop, not just recon:

1. **Recon** - subdomain enumeration (6+ tools in parallel), CRT, AlienVault OTX, VirusTotal, CIDR expansion, Wayback Machine
2. **URL collection** - `gau` + `waybackurls` + `hakrawler` + `gospider` + `katana` pipelines, httpx triage
3. **Testing** - 403 bypass, PUT method, Firebase, JWT, fullwidth / punycode, clickjacking PoCs
4. **Method** - P1 hunting methodology, scope filters, AI research prompts, report templates

If you are new, start with `08-Methodology/` then `12-Installation/`, then run `99-Misc/boomer-all-in-one.sh`.

## Quick Start

```bash
# 1. Install recon stack (Ubuntu / Kali)
bash 12-Installation/install-tools.sh
# idempotent Go + Python installer:
bash 12-Installation/install-tools2.sh
# Ars0n framework + API key manager:
bash 12-Installation/install-tools3.sh

# 2. All-in-one recon (12 sub-tools in one script)
bash 99-Misc/boomer-all-in-one.sh -h

# 3. Subdomain enumeration
bash 01-Recon-Subdomains/subdom.sh example.com
bash 01-Recon-Subdomains/alldomz.sh example.com
python3 01-Recon-Subdomains/crtsh.py -d example.com

# 4. Collect URLs
bash 02-URL-Collection/all-urls.sh example.com

# 5. Follow the methodology
cat 08-Methodology/methodology-overview.txt
cat 08-Methodology/hunting-methodology.md
```

Requirements: `bash`, `curl`, `jq`, `python3`, `go` (for installers). Each script checks its own dependencies where needed.

## Contents

| # | Folder | Count | Purpose |
|---|--------|-------|---------|
| 01 | `01-Recon-Subdomains/` | 13 | Subdomain enumeration & passive recon |
| 02 | `02-URL-Collection/` | 5 | URL gathering, filtering & sorting |
| 03 | `03-Testing-Attacks/` | 9 | Exploitation scripts & PoC generators |
| 04 | `04-Email-Tools/` | 3 | Email enumeration & normalization testing |
| 05 | `05-Utility-Scripts/` | 6 | Sorting, downloading, tech detection |
| 06 | `06-Burp-Extensions/` | 4 | Burp Suite plugins |
| 07 | `07-Bookmarklets/` | 10 | Browser-based recon bookmarklets |
| 08 | `08-Methodology/` | 4 | Hunting patterns, guides, scope filters |
| 09 | `09-AI-Prompts/` | 3 | Prompts for AI-assisted bug hunting |
| 10 | `10-Wordlists/` | 9 | Regex patterns, dorks, ports, keywords |
| 11 | `11-Report-Templates/` | 2 | Bug bounty report formatting templates |
| 12 | `12-Installation/` | 4 | Environment & tool setup scripts |
| 99 | `99-Misc/` | 4 | Legacy scripts & misc |
| | **Total** | **76** | |

### Directory Tree

```
bug-hunter-toolkit/
├── README.md
│
├── 01-Recon-Subdomains/           Subdomain & passive recon
│   ├── alldomz.sh                 Run 6 subdomain enumerators in parallel
│   ├── crtsh.py                   CRT.sh + CertSpotter subdomain fetcher
│   ├── subdom.sh                  Fetch subdomains from 25+ sources
│   ├── alienvault-urls.sh         AlienVault OTX URL scraper
│   ├── virustotal-urls.sh         VirusTotal v2 URL fetcher (API key rotation)
│   ├── cidr-to-ips.sh             Expand CIDR ranges to individual IPs
│   ├── cidr-to-live-domains.sh    CIDR → IP → reverse DNS → live domains
│   ├── domain-to-ip.sh            Resolve domains to IP addresses
│   ├── wayback-machine.sh         Wayback Machine URL + interesting files fetcher
│   ├── waybackmachine-shortcuts.txt  Wayback CDX API query collection
│   ├── zipfinder.sh               Find backup/archive files on Wayback Machine
│   ├── wayback-data-extensions.txt   Wayback query for sensitive file types
│   └── wayback-data-normal.txt    Standard Wayback CDX URL query
│
├── 02-URL-Collection/             URL gathering & filtering
│   ├── all-urls.sh                Pipe domain through gau, waybackurls, hakrawler, gospider, katana
│   ├── oneliner-urls.sh           Same toolchain, one-liner variant
│   ├── httpx-status-split.sh      Split httpx output by HTTP status code
│   ├── exclude-images.sh          Filter out image URLs from a url list
│   └── download-directory-listing.sh  Download all files from open directory listing
│
├── 03-Testing-Attacks/            Exploitation & PoC scripts
│   ├── 403-bypass.sh              HTTP 403/401 bypass via headers, methods, protocols
│   ├── put-method.sh              Check & exploit HTTP PUT method for file upload
│   ├── firebase-write-test.py     Test insecure Firebase database write permissions
│   ├── jwt-modifier.py            Decode, modify payload, re-encode JWT tokens
│   ├── fullwidth-char.py          ASCII → fullwidth Unicode (WAF bypass)
│   ├── punycode-gen.py            Homoglyph/punycode domain variants generator
│   ├── common-case-change.sh      All upper/lowercase permutations of a string
│   ├── clickjacking-poc.html      Clickjacking PoC with credential capture → Discord
│   └── realistic-clickjacking.html  Clickjacking PoC with reward overlay → Discord
│
├── 04-Email-Tools/                Email enumeration & testing
│   ├── email-finder.sh            Scrape emails from skymem.info
│   ├── email-case-sensitivity.sh  Case-varied email permutations
│   └── email-dot-variation.sh     Dot-variation email permutations
│
├── 05-Utility-Scripts/            Misc utilities
│   ├── sort-data.sh               Sort lines by string length
│   ├── sort-by-line-length.sh     Sort 200s.txt by line length + deduplicate
│   ├── random-text-generator.sh   Generate 100 random alphanumeric lines
│   ├── mass-github-repo-download.sh  Clone all repos from a GitHub org
│   ├── reconroyale-validator.py   Validate subdomains DNS + count Recon Royale points
│   └── wappalyzer.py              Detect web tech stacks (with versions)
│
├── 06-Burp-Extensions/            Burp Suite plugins
│   ├── burp-extension-final.py    Categorize requests by URL pattern (with dedup)
│   ├── burp-extension-simple.py   Categorize requests by URL pattern (basic)
│   ├── cspt-plugin.zip            CSP Toolkit plugin
│   └── cspt-plugin-updated.zip    CSP Toolkit plugin (updated)
│
├── 07-Bookmarklets/               Browser bookmarklets for recon
│   ├── alienvault-otx.js          Open AlienVault OTX for current domain
│   ├── clickjacking-test.js       Test page via web.clickjacker.io
│   ├── endpoint-finder.js         Extract API endpoints from JS + page
│   ├── hidden-fields.js           Reveal hidden/disabled form fields
│   ├── js-finder.js               Find JS file URLs in page source
│   ├── param-finder.js            Extract form parameter names + modal
│   ├── path-finder.js             Crawl & discover URL paths on page
│   ├── securitytrails-extractor.js  Extract all subdomains across SecurityTrails pages
│   ├── discord-notes-sender.js    Floating notes popup with Send to Discord (from Single-Script repo)
│   └── notion-notes-sender.js     Draggable notes popup with Send to Notion (from Single-Script repo)
│
├── 08-Methodology/                Hunting methodology & guides
│   ├── hunting-methodology.md     5 P1 vulnerability patterns with examples
│   ├── methodology-overview.txt   Recon cheatsheet (JS, dns, dir brute, GitHub)
│   ├── reverse-engineering-prompt.md  Thick client/macOS/Android reversing methodology
│   └── scope-restriction.md       What to report vs skip (P1/P2/P3 filter)
│
├── 09-AI-Prompts/                 AI-assisted hunting prompts
│   ├── blackbox-research-prompt.txt   Generate 20 business logic vulns for any target
│   ├── opencode-enumeration.txt       Automated subdomain → triage → probe pipeline
│   └── structure.md               macOS binary bug bounty session structure
│
├── 10-Wordlists/                  Wordlists, regexes & dorks
│   ├── 1592-regex-patterns.txt    1592 regex patterns for 150+ API key types
│   ├── all-ports.txt              ~300 common TCP ports for scanning
│   ├── developer-tools-regex.txt  Sensitive keyword regex collection
│   ├── github-dorks.txt           GitHub subdomain + secret dork patterns
│   ├── github-security-programs.md  GitHub dorks for finding security@ emails
│   ├── github-subdomain-dorks.md  GitHub dorks for subdomain discovery
│   ├── resolvers.txt              Public DNS resolvers (Cloudflare, Google, Quad9...)
│   ├── sensitive-extensions.txt   Common sensitive file extensions
│   └── sensitive-keywords.txt     2300+ sensitive keyword patterns
│
├── 11-Report-Templates/           Bug bounty report templates
│   ├── report-template.md         Full bug bounty report with CVSS breakdown
│   └── raw-report-template.md     Minimal format report template
│
├── 12-Installation/               Tool installation scripts
│   ├── install-tools.sh           Install 30+ recon/security tools
│   ├── install-tools2.sh          Install Go + Python recon tools (idempotent)
│   ├── install-tools3.sh          Ars0n Framework installer + API key manager
│   └── boomer-requirements.sh     Install dependencies for Boomer.sh
│
└── 99-Misc/                       Miscellaneous / legacy
    ├── boomer-all-in-one.sh       All-in-one: 12 integrated recon sub-tools
    ├── alldomz-v1.sh              Original all-domains script (parallel)
    ├── alldomz-v2.sh              All-domains one-liner variant
    └── randomscript.js            GTA typing cheat script
```

## Highlights

**Recon that actually scales**
- `subdom.sh` (25+ sources), `alldomz.sh` (parallel enumerators), `crtsh.py`, OTX + VirusTotal scrapers, CIDR → live domains, Wayback fetchers + CDX shortcut library.

**URL pipeline**
- `all-urls.sh` / `oneliner-urls.sh` chain gau, waybackurls, hakrawler, gospider, katana. Split by status with `httpx-status-split.sh`.

**Testing Cheat-codes**
- 403 bypass (181KB header/method/protocol matrix), PUT upload check, Firebase write test, JWT modifier, WAF bypass via fullwidth chars, punycode homoglyphs.

**Browser edge**
- 10 bookmarklets: endpoint / param / path / JS finder, hidden fields revealer, SecurityTrails extractor, OTX opener, clickjacking tester, plus Discord and Notion quick-note senders.

**Method + AI**
- `hunting-methodology.md`: 5 chained P1 patterns from real reports. `scope-restriction.md`: what to skip. `blackbox-research-prompt.txt` + `opencode-enumeration.txt` for AI-assisted hunting.

**Wordlists**
- 1592 regexes for secrets, 2300+ sensitive keywords, GitHub dorks, resolvers, ports, extensions.

## Usage Notes

- Make scripts executable: `chmod +x 01-Recon-Subdomains/*.sh 02-URL-Collection/*.sh`
- Most `.sh` take a domain or file: `bash script.sh example.com` or `bash script.sh domains.txt` - check header comments (`-h` where supported).
- Python scripts need `python3` + `requests` in most cases: `pip3 install requests`.
- Burp extensions: Extender → Add → Python → load `06-Burp-Extensions/burp-extension-final.py` (needs Jython).
- Bookmarklets: create new bookmark → paste file contents as URL → click on target page.
- VirusTotal script needs your own API keys - replace `key-1/2/3` placeholders.
- Clickjacking PoCs post to Discord webhook - replace webhook URL before use.

## Legal & Responsible Use

For **educational purposes and authorized testing only**. Only test targets covered by a bug bounty scope, VDP, contract, or your own assets. The author assumes no liability for misuse.

If you find a vulnerability: stop, document, report via the program's channel, do not exfiltrate or pivot. Use `11-Report-Templates/` for clean reports.

## Contributing

PRs welcome for bug fixes, new sources, and deduped wordlists. Keep scripts POSIX-friendly, add `usage()` + `-h`, and don't commit API keys, webhooks, or target data.

## License

No license file yet - if you want reuse, add MIT. Until then, all rights reserved to the author, viewing/forking for learning only.

## Author

Maintained by [@Raunaksplanet](https://github.com/Raunaksplanet) - bug bounty hunter. Tooling built from live hunting sessions.
