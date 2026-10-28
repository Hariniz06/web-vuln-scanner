Web Application  Vulnerability Scanner

A Python-based web application security assessment tool that crawls a target web application, discovers pages and input points, runs payload-based checks for selected vulnerability classes, reviews HTTP security headers, and generates a structured assessment report.
Use only on applications you own or have explicit permission to test. This scanner sends active test requests and may modify application state or trigger security controls. Use a deliberately vulnerable lab such as DVWA or a staging application with written authorization.

Overview
The scanner is implemented in ethical-pentest-scanner.py. It uses HTTP requests and HTML parsing to identify reachable pages, forms, and parameters, then tests discovered inputs with predefined security payloads. Findings are collected with details such as the affected URL, parameter, evidence, severity, confidence, remediation guidance, CWE/OWASP mapping, and risk scoring where available.

The project is intended for cybersecurity learning, authorized penetration-testing exercises, and preliminary security reviews. It is not a replacement for a professional penetration test or a guarantee that an application is secure.
Features
The implementation includes checks and discovery logic for the following areas:
Area	What the scanner checks
Target reconnaissance	Collects basic server, technology, response, header, cookie, and TLS-related information where available
Crawling and discovery	Visits pages up to a configurable crawl depth and discovers links, forms, and input parameters
SQL injection (SQLi)	Sends boolean-, union-, error-, time-based-, and stacked-query-style test payloads and looks for response indicators
Cross-site scripting (XSS)	Tests reflected input with common script/HTML payloads and checks whether payloads appear executable in the response context
Server-side request forgery (SSRF)	Tests URL-like parameters with internal-address and alternate-scheme payloads and looks for response indicators
Insecure direct object references (IDOR)	Tries alternate identifier values and compares responses for potentially unauthorized object access
Command injection	Sends Unix/Linux and Windows command-style payloads and checks for indicators in responses
Path traversal	Tests file/path parameters with traversal patterns and checks for recognizable file-content indicators
CSRF indicators	Examines discovered forms for common anti-CSRF token names and related indicators
XXE	Submits XML external-entity test payloads to discovered form actions and looks for response indicators
HTTP security headers	Checks the target response for important browser-facing security headers
Reporting	Produces a JSON report or a detailed text report with summary statistics, findings, and remediation recommendations


Important: These are active, heuristic checks. A listed test category does not mean every variant of that vulnerability is comprehensively detected or reliably confirmed.
How It Works
The scan runs through these main phases:
1. Target normalization
   The supplied target is normalized to a URL. If the scheme is omitted, the scanner assumes http://.
2. Reconnaissance and fingerprinting
   The scanner requests the target and collects available information such as response headers, cookies, server/technology hints, response timing, and TLS-related details.
3. Crawling and input discovery
   It crawls reachable pages up to the selected depth, parses HTML with Beautiful Soup, and records discovered links, forms, and GET/POST input parameters.
4. Active vulnerability checks
   For discovered GET/POST parameters, the scanner schedules SQLi, XSS, SSRF, IDOR, command-injection, and path-traversal checks. It also checks discovered forms for CSRF-related indicators and XXE behavior, and checks the target for HTTP security headers.
5. Concurrent request handling
   Vulnerability checks are run using a thread pool. Requests include a timeout and a short delay; the scanner also rotates among predefined browser-like User-Agent strings.
6. Finding analysis and deduplication
   Responses are inspected for signals associated with each test. Findings are deduplicated and enriched with severity, confidence, evidence, remediation suggestions, and risk/compliance metadata where the implementation provides them. These signals can produce false positives or miss vulnerabilities.
7. Report generation
   The scanner prints a console summary and saves the requested JSON or detailed text report.
   
Technology Stack
- Python 3
- Requests — HTTP communication
- Beautiful Soup 4 — HTML parsing and form/link discovery
- lxml — HTML/XML parsing support
- dnspython — optional DNS-related functionality
- colorama — optional colored console output
- Python standard library modules for concurrency, URL handling, TLS/socket inspection, JSON serialization, and timestamps
  
Requirements
- Python 3.9 or later is recommended.
- Network access to the authorized target.
- Permission to test the target.
- Install the dependencies below. dnspython and colorama are treated as optional by the script, but installing them enables their associated functionality.
Installation
Clone the repository and enter the project directory:
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd <YOUR_REPOSITORY_DIRECTORY>
Create and activate a virtual environment.
Windows PowerShell:
python -m venv .venv
.\.venv\Scripts\Activate.ps1
Linux / Kali Linux / macOS:
python3 -m venv .venv
source .venv/bin/activate
Install dependencies:
python -m pip install --upgrade pip
python -m pip install requests beautifulsoup4 lxml dnspython colorama

Usage
Run a basic scan against an authorized target:
python ethical-pentest-scanner.py --target http://127.0.0.1/dvwa
Set the crawl depth and output path:
python ethical-pentest-scanner.py --target http://127.0.0.1/dvwa --depth 3 --output reports/dvwa_scan.json
Generate the detailed text report:
python ethical-pentest-scanner.py --target http://127.0.0.1/dvwa --report-format detailed --output reports/dvwa_scan.json
The detailed format writes a .txt file by replacing the .json suffix in the output path. For example, the command above writes reports/dvwa_scan.txt.

Enable verbose error information:
python ethical-pentest-scanner.py --target http://127.0.0.1/dvwa --verbose
Suppress most console output:
python ethical-pentest-scanner.py --target http://127.0.0.1/dvwa --quiet
Show available command-line options:
python ethical-pentest-scanner.py --help

Command-line options
Option	Description	Default
--target	Target application URL; required	None
--depth	Maximum crawl depth used for discovery	5
--output	Report output path	reports/ethical_pentest_report.json
--report-format	json or detailed	json
--quiet	Suppresses normal console output	Off
--verbose	Prints additional error details	Off


For a first run, use a local test lab and a shallow depth. Review the target and test plan before running active checks.

Output and Reports
By default, the scanner writes:
reports/
└── ethical_pentest_report.json
The JSON report contains these top-level sections:
- scan_metadata — target, start/end times, scan duration, and scanner version
- target_information — collected target fingerprint and response details
- executive_summary — overall counts, severity distribution, risk level, and related summary metrics
- compliance_status — implementation-defined OWASP/PCI-DSS mapping status
- statistics — crawl, form, parameter, request, payload, and finding counters
- findings — individual findings and their evidence/remediation fields
- recommendations — generated security improvement suggestions
Each finding can include fields such as vulnerability type, severity, confidence, target URL, parameter, description, evidence, payload used, request/response details, remediation, references, CWE ID, OWASP category, CVSS score, risk score, business/technical impact, and compliance impact.

Treat reports as sensitive: they can contain URLs, parameter values, request/response excerpts, and other application details. Do not commit reports from real systems to a public repository without reviewing and sanitizing them.

Understanding Findings
- Severity and risk scores are estimates produced by the scanner's logic, not an independent validation or a formal guarantee of exploitability.
- Evidence matters. Review the recorded URL, parameter, request/response, and payload before deciding whether a finding is real.
- False positives are possible. Generic error messages, reflected input, application behavior, and network delays may resemble vulnerability indicators.
- False negatives are possible. Authentication requirements, JavaScript-heavy pages, custom workflows, CSRF protections, rate limits, WAFs, and application-specific behavior can prevent discovery or detection.
- Re-test findings manually in a safe environment and document validation steps before treating them as confirmed vulnerabilities.
  
Limitations and Safety Notes
- The scanner primarily discovers inputs through HTTP crawling and parsed HTML. It is not a full browser and does not execute JavaScript like a modern browser.
- Checks are based on predefined payloads and response heuristics; they may not cover all application frameworks, APIs, authentication flows, or vulnerability variants.
- Authenticated scanning and complex multi-step workflows may require additional configuration or code changes.
- The current request helper disables TLS certificate verification (verify=False) and suppresses related unverified-HTTPS warnings. This can hide certificate problems and should be corrected before using the tool in a production-quality workflow.
- The script uses active payloads. Some payload strings can be intrusive or destructive if they reach a vulnerable system. Do not run against production or third-party systems without explicit authorization and a reviewed test plan.
- Concurrent checks can generate noticeable traffic. Reduce the scope/depth and avoid running multiple scans at once on fragile systems.
- A successful scan does not prove the target is secure. Use this tool as one input to a broader review.
  
Project Structure
.
├── ethical-pentest-scanner.py
└── reports/
    └── ethical_pentest_report.json


Responsible Use
This project is intended for education and authorized security assessment only.
- Test only systems you own or have explicit written permission to assess.
- Prefer intentionally vulnerable labs such as DVWA, local containers, or isolated staging environments.
- Do not use the scanner to access, alter, or extract data without authorization.
- Follow the scope, rate limits, and disclosure requirements agreed with the system owner.
- Securely store reports and remove sensitive information before sharing them.
