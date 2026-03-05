---
name: Bug Bounty Hunter
version: 1.0.0
description: Automated security vulnerability discovery and reward management system for bug bounty programs
author: Security Ops Team
tags:
  - bug bounty
  - security
  - hunting
  - vulnerability
  - reconnaissance
dependencies:
  - python >= 3.9
  - nmap >= 7.0
  - nuclei >= 2.9.0
  - sqlmap >= 1.7
  - subfinder >= 2.6
  - httpx >= 1.4
  - waymore >= 2.0
  - dalfox >= 2.0
  - jsluice
  - grep.app
  - amass
  - massdns
---

## Purpose

The Bug Bounty Hunter skill automates the complete workflow of discovering, verifying, and reporting security vulnerabilities to earn bug bounty rewards. It integrates with HackerOne, Bugcrowd, and OpenBugBounty platforms to maximize efficiency and minimize manual effort.

Real use cases:
1. **Target reconnaissance**: Enumerate subdomains, endpoints, JS files, and exposed services for a given scope
2. **Vulnerability detection**: Run automated scans for XSS, SQL injection, SSRF, IDOR, misconfigurations, and information disclosure
3. **Proof-of-concept generation**: Create reproducible exploit chains with captured HTTP requests and screenshots
4. **Platform integration**: Submit findings directly to bug bounty platforms with correctly formatted reports
5. **Duplicate detection**: Check for existing reports before submission using platform APIs and public databases
6. **Bounty tracking**: Monitor payout status, communicate with security teams, and track earnings across programs
7. **Continuous hunting**: Schedule periodic scans for new assets and emerging vulnerabilities
8. **Scope validation**: Ensure all testing stays within authorized boundaries using `--check-scope` validation

## Scope

Commands:
- `enumerate <target>`: Discover subdomains, endpoints, JS files, and historical URLs
- `scan <targets_file>`: Run automated vulnerability scans with configurable tools and templates
- `test <target> <vuln_type>`: Manual verification with custom payloads and PoC generation
- `report <findings_file>`: Generate platform-specific report ready for submission
- `claim <report_file> [--platform <name>]`: Submit report via API and initiate bounty claim
- `track [--report-id <id>]`: Monitor status, bounty amount, and payment of reports
- `verify <report-id>`: Re-test fixed vulnerabilities to close the loop
- `config [key] [value]`: Manage API tokens, rate limits, and tool configurations
- `scope validate <target>`: Check if target is within authorized scope
- `duplicates check <vulnerability>`: Search public databases for existing reports

## Detailed Work Process

### 1. Target Enumeration & Scope Validation
```bash
$ bug-bounty-hunter enumerate --target example.com --output assets.json --include-subdomains --js
[+] Starting enumeration for example.com
[+] Running subfinder... 156 subdomains discovered
[+] Probing with httpx... 89 live hosts
[+] Extracting JS endpoints... 34 files found
[+] Harvesting historical URLs with waymore... 2,450 endpoints
[+] Assets saved to assets.json
```
- Uses `subfinder` for passive subdomain discovery
- `httpx` probes for live services and technologies
- `waymore` collects historical URLs from Wayback Machine and Common Crawl
- `jsfinder` extracts endpoints and secrets from JavaScript files
- `amass` for deeper DNS enumeration (configurable)

### 2. Vulnerability Scanning
```bash
$ bug-bounty-hunter scan --targets assets.json --tools nuclei,sqlmap,dalfox --severity high,critical --rate-limit 5
[+] Loading nuclei templates (2,400+ templates)
[+] Testing 89 targets...
[+] Nuclei: 12 potential XSS, 3 SSRF, 1 SQLi found
[+] SQLmap: Confirmed 1 boolean-based SQLi in /product.php?id=
[+] Dalfox: 5 reflected XSS in search parameters
[+] Findings saved to findings_20250115.json
```
- `nuclei` with custom template selection (`-templates cves/,xss/,ssrf/`)
- `sqlmap` with `--batch` for automated exploitation
- `dalfox` for XSS with `--found-callback` for payload confirmation
- `massdns` for subdomain takeover checks
- Rate limiting configurable per tool to avoid service disruption

### 3. Manual Verification & PoC Generation
```bash
$ bug-bounty-hunter test --target "https://example.com/search?q=" --vulnerability xss --payload '<svg onload=alert(document.domain)>' --screenshot
[+] Testing XSS in search parameter
[+] Payload delivered: <svg onload=alert(document.domain)>
[+] Alert triggered with domain: example.com
[+] Screenshot captured: xss_proof_20250115_143022.png
[+] HTTP request logged: xss_request_20250115_143022.har
[+] Confirmed: Reflected XSS in search endpoint
```
- Uses `dalfox` headless browser for DOM-based XSS
- `grep.app` for searching sensitive data leaks in code
- `jsluice` for API key extraction from JS files
- Automatic HAR capture for network reproduction

### 4. Duplicate Detection
```bash
$ bug-bounty-hunter duplicates check --vulnerability "XSS in search parameter" --target example.com
[+] Searching HackerOne public reports...
[+] No duplicates found on HackerOne
[+] Searching Bugcrowd disclosures...
[+] 1 similar report found on Bugcrowd (different endpoint)
[+] Verifiable as unique: Yes
```
- Queries platform API with scoped authentication
- Searches public disclosure databases (H1, Bugcrowd, OpenBugBounty)
- Compares endpoint, vulnerability type, and impact

### 5. Report Generation
```bash
$ bug-bounty-hunter report --findings findings_20250115.json --platform hackerone --program example-com
[+] Generating HackerOne formatted report...
[+] Impact assessment: Medium (XSS leads to session hijacking)
[+] CVSS 3.1 Score: 6.1
[+] Steps to reproduce:
    1. Navigate to https://example.com/search
    2. Search for: <svg onload=alert(1)>
    3. Alert box appears with domain
[+] Report saved: h1_report_20250115.json
[+] Includes: 3 screenshots, HAR file, curl command
```
- Platform-specific templates (H1, Bugcrowd, OBB)
- Automatic CVSS scoring based on vulnerability type
- Includes `curl` command for reproduction
- Attaches evidence (screenshots, network logs)

### 6. Bounty Claim
```bash
$ bug-bounty-hunter claim --report h1_report_20250115.json --platform hackerone --program "example-com"
[+] Authenticating with HackerOne...
[+] Submitting report to program: example-com
[+] Report submitted successfully
[+] Report ID: https://hackerone.com/reports/1234567
[+] Tracking ID: BH-2025-001
[+] Estimated bounty: $500-$1,500 (Medium severity)
[+] Notification sent to Slack channel #bug-bounties
```
- Uses platform API with stored credentials
- Handles two-factor authentication via stored session tokens
- Sets up webhook notifications for status updates

### 7. Tracking & Management
```bash
$ bug-bounty-hunter track --last-30-days
[+] Recent reports:
ID               | Target         | Vulnerability   | Status    | Bounty
-----------------|----------------|-----------------|-----------|----------
BH-2025-001      | example.com    | XSS             | Triaged   | $750
BH-2025-002      | test.com       | SQLi            | Resolved  | $1,200
BH-2025-003      | demo.org       | IDOR            | Duplicate| $0

Total earned YTD: $12,450
```
- Displays all reports with status and bounty amounts
- Exports to CSV for tax purposes
- Sends weekly summary emails

### 8. Fix Verification
```bash
$ bug-bounty-hunter verify --report-id BH-2025-001
[+] Re-testing vulnerability from report BH-2025-001
[+] Target: https://example.com/search
[+] Payload: <svg onload=alert(1)>
[+] Status: Fixed (no alert triggered)
[+] Vulnerability no longer reproducible
[+] Updating report status to "Resolved"
[+] Notifying HackerOne team
```
- Re-runs original PoC against target
- Verifies patch deployment
- Updates platform status automatically

## Golden Rules

1. **Authorization First**: Only test targets listed in authorized bug bounty programs. Use `bug-bounty-hunter scope validate <target>` before scanning.
2. **Rate Limiting**: Never exceed 10 requests/second unless explicitly permitted. Default: `--rate-limit 5`. Use `--throttle 2000` for 2s delays.
3. **No Data Exfiltration**: Do not download, modify, or delete data. Proof-of-concept only. Use `--read-only` mode when available.
4. **Platform Scope Compliance**: Each platform has rules. Run `bug-bounty-hunter config get platform-rules` before scanning.
5. **No DDoS/Resource Exhaustion**: Avoid flooding endpoints with payloads. Limit `--concurrent 5` by default.
6. **Evidence Only**: Capture screenshots, HTTP requests, curl commands. Do NOT execute destructive payloads (rm, drop table, etc.).
7. **Responsible Disclosure**: Never publish before vendor patch. Use platform coordination channels.
8. **Accurate Reporting**: Report duplicates honestly. Do not claim rewards for known issues.
9. **Data Privacy**: Delete collected PII after report submission. Use `--purge-data` flag after claim.
10. **Legal Compliance**: Some countries restrict security research. Check local laws before proceeding.

## Examples

**Example 1: Reconnaissance on new target**
```bash
$ bug-bounty-hunter enumerate --target "*.uber.com" --output uber_assets.json --threads 20 --exclude-wildcard
[+] Scope: *.uber.com (authorized via HackerOne)
[+] Subfinder: 1,247 subdomains
[+] Live hosts: 734
[+] JS files: 156 containing API keys
[+] Exported: uber_assets.json (54 MB)
```

**Example 2: Targeted SQL injection scan**
```bash
$ bug-bounty-hunter scan --targets uber_assets.json --tool sqlmap --level 5 --risk 3 --dbs --threads 3 --output sqlmap_results/
[+] Testing 734 live hosts
[+] Parameter discovery complete
[+] Database enumeration on 3 targets:
    - https://api.uber.com/coupons (MySQL)
    - https://riders.uber.com/profile (PostgreSQL)
    - https://partners.uber.com/earnings (SQL Server)
[+] SQLi confirmed: api.uber.com/coupons?id=1
[+] Extracted: 4 databases, 12 tables, 450+ rows
[+] Saved: sqlmap_results/api.uber.com/
```

**Example 3: XSS verification with PoC**
```bash
$ bug-bounty-hunter test --target "https://partners.uber.com/earnings?date=" --vulnerability xss --payload "<img src=x onerror=alert(document.domain)>" --screenshot --har
[+] XSS test on partners.uber.com/earnings
[+] Payload executed: alert(document.domain) = partners.uber.com
[+] Session cookies accessible via document.cookie
[+] Impact: Session hijacking possible
[+] Evidence: screenshot_20250115_162233.png, request_20250115_162233.har
[+] Reproducible: Yes (Chrome 120, Firefox 121)
[+] CVSS:6.1 (Medium)
```

**Example 4: Submit to Bugcrowd**
```bash
$ bug-bounty-hunter claim --report xss_report_uber.json --platform bugcrowd --program "uber"
[+] Bugcrowd authentication: OK
[+] Program: Uber (ID: 5678)
[+] Submission valid: Yes (in scope)
[+] Report submitted: https://bugcrowd.com/uber/reports/9876543
[+] Estimated bounty: $750-$2,000
[+] Slack notification sent to #uber-bounties
```

**Example 5: Track earnings**
```bash
$ bug-bounty-hunter track --platform hackerone --resolved --csv
Report ID,Target,Vulnerability,Status,Bounty,Paid
BH-2024-045,api.uber.com,SQLi,Resolved,$1,500,$1,500
BH-2024-089,riders.uber.com,IDOR,Duplicate,$0,$0
BH-2025-012,partners.uber.com,XSS,Resolved,$750,$750
BH-2025-023,m.uber.com,CSRF,Triaged,$500,$0
Total: $2,250 paid, $500 pending
```

## Rollback Commands

- `bug-bounty-hunter scan stop --all`: Abort all running scans immediately
- `bug-bounty-hunter purge --findings <date>`: Delete findings older than specified date (GDPR compliance)
- `bug-bounty-hunter claim withdraw <report-id>`: Retract submission before acceptance (H1 only)
- `bug-bounty-hunter config reset --tokens`: Clear all stored API tokens and credentials
- `bug-bounty-hunter track delete <report-id>`: Remove report from tracking database (use with caution)
- `bug-bounty-hunter enumerate cleanup <target>`: Remove all enumeration artifacts for given target
- `bug-bounty-hunter test revert <test-id>`: Undo any state changes from manual testing (if applicable)
- `bug-bounty-hunter logs clear --before <timestamp>`: Clear logs older than timestamp for privacy

## Dependencies

Required tools (auto-detected on PATH):
```
nmap         - Network scanning and service detection
nuclei       - Vulnerability scanner with YAML templates
sqlmap       - Automated SQL injection and database takeover
subfinder    - Subdomain enumeration
httpx        - HTTP probe and service detection
waymore      - Historical URL collection from Wayback & Common Crawl
dalfox       - XSS scanning and parameter analysis
amass        - Advanced DNS enumeration (optional but recommended)
massdns      - DNS brute-forcing and resolution
jsluice      - JavaScript parsing for endpoint extraction
grep.app     - Sensitive data search in public repos (API token scraper)
```

Installation:
```bash
# Install Go tools
go install -v github.com/projectdiscovery/nuclei/v2/cmd/nuclei@v2.9.0
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@v2.6.0
go install -v github.com/projectdiscovery/httpx/cmd/httpx@v1.4.0
go install -v github.com/owasp-amass/amass/v3/...@master

# Install Python tools
pip3 install sqlmap dalfox jsluice waymore

# Install bug-bounty-hunter skill
git clone https://github.com/bug-bounty-bot/skill ~/.bug-bounty-hunter
echo 'export PATH="$HOME/.bug-bounty-hunter/bin:$PATH"' >> ~/.bashrc
```

Configuration:
```bash
# Initialize configuration
bug-bounty-hunter config init

# Set platform tokens (read from env or set manually)
bug-bounty-hunter config set hackerone.token $HACKERONE_TOKEN
bug-bounty-hunter config set bugcrowd.token $BUGCROWD_TOKEN
bug-bounty-hunter config set openbugbounty.token $OPENBUGBOUNTY_TOKEN

# Configure tool-specific settings
bug-bounty-hunter config set nuclei.templates ~/nuclei-templates/
bug-bounty-hunter config set sqlmap.threads 3
bug-bounty-hunter config set rate-limit 5
bug-bounty-hunter config set max-concurrent-scans 2

# Add authorized targets (scope validation)
bug-bounty-hunter scope add --target "*.uber.com" --platform hackerone --program "uber"
```

## Verification

Verify installation:
```bash
$ bug-bounty-hunter --version
Bug Bounty Hunter 1.0.0

$ bug-bounty-hunter tools check
nmap:          v7.94 (✓)
nuclei:        v2.9.0 (✓)
sqlmap:        v1.7.4 (✓)
subfinder:     v2.6.0 (✓)
httpx:         v1.4.0 (✓)
waymore:       v2.0 (✓)
dalfox:        v2.0 (✓)
jsluice:       v0.1.6 (✓)
All dependencies installed successfully.
```

Verify API connectivity:
```bash
$ bug-bounty-hunter platforms test
Testing Hackerone...  Authenticated (user: security_researcher)
Testing Bugcrowd...   Authenticated (user: @archomboldt)
Testing OpenBugBounty...   Not configured
All configured platforms reachable.
```

Test on safe target:
```bash
# Run against testphp.vulnweb.com (intentionally vulnerable)
$ bug-bounty-hunter enumerate --target testphp.vulnweb.com --output test_assets.json
$ bug-bounty-hunter scan --targets test_assets.json --tool nuclei --severity all
[+] Found XSS in /search.php?test= parameter
[+] Found SQLi in /listproducts.php?cat= parameter
[+] Found LFI in /include.php?file= parameter
# Expected: 3+ vulnerabilities on test site
```

## Troubleshooting

**Issue: "Command not found: bug-bounty-hunter"**
Solution: Ensure `~/.bug-bounty-hunter/bin` is in PATH. Run `source ~/.bashrc` or logout/login.

**Issue: "nuclei: template not found"**
Solution: Update templates: `nuclei -update-templates`. Set custom path: `bug-bounty-hunter config set nuclei.templates ~/custom-templates/`.

**Issue: "Rate limited by target"**
Solution: Reduce concurrency: `bug-bounty-hunter config set rate-limit 2`. Add delays: `bug-bounty-hunter config set throttle 5000`. Check if IP blocked.

**Issue: "Authentication failed on platform API"**
Solution: Verify token permissions: must have `report:create` and `target:read`. Regenerate token on platform dashboard. Ensure token not expired.

**Issue: "SQLmap hangs or times out"**
Solution: Reduce `--level` and `--risk`. Use `--time-sec 10`. Add `--tamper=space2comment`. Check for WAF blocking.

**Issue: "False positives from nuclei"**
Solution: Use specific templates: `-templates cves/,xss/`. Add `-exclude-templates Dos/,PathTraversal/`. Manually verify before submission.

**Issue: "Duplicate report rejected"**
Solution: Check platform scope rules; some prohibit certain tools. Search public databases thoroughly before submission. Some programs have private scope—use platform UI to verify.

**Issue: "Cannot claim bounty (program not found)"**
Solution: Verify program ID/spelling. Some programs require invitation. Check if program is open to all researchers or invitation-only.

**Issue: "No subdomains discovered"**
Solution: Try `amass` instead of `subfinder`. Add `--passive-only false` for DNS brute-forcing. Check target spelling and scope.

**Issue: "JavaScript files not parsed"**
Solution: Install `jsluice`: `pip3 install jsluice`. Verify files are JavaScript (not minified CSS/HTML). Use `--extract-secrets` flag.

**Issue: "Memory errors during scan"**
Solution: Reduce `--threads`. Split target list into chunks. Use swap file: `sudo swapon -a`. Monitor with `top`.

## Security & Ethics

- Always obtain explicit authorization before scanning
- Never test out-of-scope targets or non-participating programs
- Do not exploit vulnerabilities beyond proof-of-concept (no data exfiltration, modification, or deletion)
- Report findings responsibly through official channels only
- Respect researcher anonymity if platform permits
- Coordinate with vendor security teams via platform
- Keep findings confidential until resolved
- Comply with bug bounty program rules and legal requirements

**Remember**: Bug bounty programs are partnerships with security teams, not adversarial penetration tests. Build trust, not disruption.

```