# OWASP Top 10: Detailed Notes with Practical Labs

## About this document

The OWASP Top 10 is a ranked list of the most critical risks in web applications. It is maintained by the Open Worldwide Application Security Project (OWASP) and is based on data from real-world security testing. This document covers the current edition (2025). Older edition names are noted where they help when you read older resources.

Each category follows the same structure:

1. **Concept:** what the risk is and why it matters
2. **How it happens:** common causes
3. **Vulnerable example:** code or configuration that shows the flaw
4. **Practical lab:** step-by-step exercise to perform on a lab you own
5. **Prevention:** the fix, with a secure example where useful
6. **Detection:** how to find it during testing or review
7. **Checklist:** what you should be able to do

Verify the official wording at https://owasp.org/Top10/ when you need the exact definitions.

---

## Lab setup (do this first)

Only test applications you own or are explicitly allowed to test. The labs below are intentionally vulnerable and should run on your own machine or VM.

### Recommended tools

| Tool | Purpose | Install |
|---|---|---|
| Docker | runs the vulnerable labs | docker.com or `sudo apt install docker.io` |
| OWASP Juice Shop | modern vulnerable app with a challenge scoreboard | see below |
| DVWA | classic vulnerable PHP app, good for basics | see below |
| Burp Suite Community | intercepting proxy for web traffic | portswigger.net/burp/communitydownload |
| Browser DevTools | inspect HTML, JavaScript, and network requests | built into Chrome and Firefox |

### Start OWASP Juice Shop

```bash
docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop
```

Open http://localhost:3000 in your browser. Stop it later with `docker stop juice-shop`.

### Start DVWA

```bash
docker run -d -p 8080:80 --name dvwa vulnerables/web-dvwa
```

Open http://localhost:8080, log in with `admin` / `password`, click "Create / Reset Database", and set the security level to **Low** under "DVWA Security".

### Configure Burp Suite

1. Start Burp Suite and go to **Proxy > Intercept** (or the Proxy tab in newer versions).
2. Set your browser proxy to `127.0.0.1:8080`. Use the FoxyProxy extension, or the browser's proxy settings.
3. Install the Burp CA certificate from http://burpsuite (while the proxy is on) so HTTPS works.
4. Browse the lab app. Every request now appears in **Proxy > HTTP history**. Use **Repeater** to resend and modify requests.

Note: DVWA and Burp both like port 8080. If Burp is already using 8080, change DVWA to another port, such as `-p 8081:80`.

---

## A01: Broken Access Control

### Concept
Access control decides who can do what. Broken access control means a user can act outside their intended permissions. It is the most common category in the Top 10.

### How it happens
- **IDOR (Insecure Direct Object Reference):** changing an ID in a URL or request (for example `/invoice/1002`) reveals another user's data.
- **Missing checks on the server:** the UI hides a button, but the API still accepts the request.
- **Privilege escalation:** changing a `role` field from `user` to `admin` in a request body.
- **Forced browsing:** admin pages exist at predictable URLs with no authorization check.
- **CORS misconfiguration** and **metadata or path manipulation** (for example `../` in file paths).

### Vulnerable example (Python, Flask)
```python
@app.route("/invoice/<int:invoice_id>")
def get_invoice(invoice_id):
    invoice = db.query("SELECT * FROM invoices WHERE id = ?", invoice_id)
    return jsonify(invoice)   # no check that the invoice belongs to the current user
```

### Practical lab (Juice Shop)
1. Register two accounts: `alice@test.com` and `bob@test.com`. Place an order as Alice.
2. Log in as Bob. Open Burp Proxy and browse to the order history or basket pages.
3. Find requests that contain IDs (such as `/rest/basket/2`). Send one to Repeater.
4. Change the ID to Alice's basket ID and resend. If you see her items, you have found an IDOR.
5. Look up the scoreboard (`/#/score-board`) and search for challenges titled "View Basket" or "Admin Section."

### Practical lab (DVWA)
1. Open the **Insecure CAPTCHA** or **Authorisation Bypass** module if available, and note how the page decides who can see content.
2. Modify a hidden form field or cookie in DevTools (such as a `role` or `admin` value) and reload.

### Prevention
- Deny by default. Every endpoint should require an explicit permission.
- Check ownership on the server for every object access:

```python
@app.route("/invoice/<int:invoice_id>")
@login_required
def get_invoice(invoice_id):
    invoice = db.query(
        "SELECT * FROM invoices WHERE id = ? AND owner_id = ?",
        invoice_id, current_user.id)
    if invoice is None:
        abort(404)   # do not reveal that the record exists
    return jsonify(invoice)
```

- Enforce access checks on the server, never only in the browser or UI.
- Use random, unpredictable identifiers (UUIDs) as an extra layer, but do not rely on them alone.
- Disable directory listing and remove unused admin routes.

### Detection
- Test every endpoint with two users of different privileges (horizontal and vertical checks).
- Try unauthenticated requests to every protected endpoint.
- Look for IDs in URLs and request bodies and change them.

### Checklist
- [ ] I can explain IDOR and find one in a lab.
- [ ] I can explain why hiding a button in the UI is not access control.
- [ ] I can write a server-side ownership check.

---

## A02: Security Misconfiguration

### Concept
Systems, frameworks, servers, and cloud services are often insecure because of default or incomplete configuration, not because of bad code.

### How it happens
- Default passwords and sample accounts left in place
- Debug mode enabled in production (stack traces leak internals)
- Unnecessary features, ports, or services enabled
- Directory listing enabled on the web server
- Overly permissive cloud storage (public buckets) or CORS policies
- Missing security headers

### Vulnerable example (Flask)
```python
app = Flask(__name__)
app.run(debug=True, host="0.0.0.0")   # debugger reachable from the network
```

### Practical lab
1. In Juice Shop, open the browser DevTools and check the response headers for the home page. Note which security headers are missing (for example `Content-Security-Policy`, `Strict-Transport-Security`).
2. Request common sensitive paths: `/robots.txt`, `/.git/`, `/ftp/`. Juice Shop exposes some of these on purpose.
3. In DVWA, look for the **Setup / Reset** page and note what is exposed.
4. Run a basic scan of your own lab container: `nmap -sV -p- localhost` and identify services you did not intend to expose.

### Prevention
- Use a repeatable, minimal hardening process for every environment.
- Remove default accounts and sample content.
- Turn off debug mode and detailed error pages in production.
- Set security headers. Example for Nginx:

```nginx
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "DENY" always;
add_header Content-Security-Policy "default-src 'self'" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

- Disable directory listing (`autoindex off;` in Nginx).
- Review cloud permissions and block public access to storage by default.

### Detection
- Compare configuration against a baseline (CIS Benchmarks).
- Scan with tools like `nikto` or `nuclei` against your lab.
- Check response headers with `curl -I http://localhost:3000`.

### Checklist
- [ ] I can list the security headers that should be present and what each does.
- [ ] I can explain why debug mode in production is dangerous.
- [ ] I can run a header check and interpret the results.

---

## A03: Software Supply Chain Failures

### Concept
Modern applications depend on many third-party libraries, build tools, containers, and CI/CD pipelines. A weakness in any of these can compromise your application. This category extends the older "Vulnerable and Outdated Components" issue.

### How it happens
- Using libraries with known vulnerabilities (CVEs)
- Pulling packages from untrusted or typo-squatted sources
- Unverified build pipelines or compromised CI/CD secrets
- Dependency confusion (a public package overrides a private package with the same name)
- Outdated base container images

### Practical lab
1. Create a small Python project with an old, known-vulnerable version:

```bash
mkdir supply-lab && cd supply-lab
python3 -m venv venv && source venv/bin/activate
pip install "requests==2.19.1"     # old version, intentionally outdated
pip list
```

2. Audit it:

```bash
pip install pip-audit
pip-audit
```

3. Also try a tool for Node projects (`npm audit`) on any Juice Shop source you download.
4. Scan a container image:

```bash
docker pull bkimminich/juice-shop
# Trivy (install from aquasecurity.github.io/trivy) scans images for known CVEs
trivy image bkimminich/juice-shop
```

### Prevention
- Keep an inventory of all components (a Software Bill of Materials, SBOM).
- Pin dependency versions and use lock files (`requirements.txt` with hashes, `package-lock.json`).
- Scan dependencies and images in your CI pipeline.
- Use official, minimal base images and verify signatures where available.
- Restrict who can publish to internal package names, and use scoped or private registries.

### Detection
- `pip-audit`, `npm audit`, `trivy`, `grype`, and Dependabot alerts on GitHub.
- Review the CI pipeline for unpinned actions and exposed secrets.

### Checklist
- [ ] I can find a known-vulnerable dependency with an audit tool.
- [ ] I can explain what a lock file does and why it matters.
- [ ] I can explain dependency confusion in one paragraph.

---

## A04: Cryptographic Failures

### Concept
Data is exposed because it is not encrypted, is encrypted with weak methods, or is handled with weak keys or protocols. This was previously named "Sensitive Data Exposure."

### How it happens
- Sending sensitive data over HTTP instead of HTTPS
- Storing passwords with fast or unsalted hashes (MD5, SHA-1, plain SHA-256)
- Using weak algorithms (DES, RC4) or deprecated TLS versions (TLS 1.0, 1.1)
- Hard-coded keys and secrets in source code
- Weak random number generators for tokens

### Vulnerable example (password storage)
```python
import hashlib
password_hash = hashlib.md5(password.encode()).hexdigest()   # fast, unsalted, broken
```

### Secure example
```python
import bcrypt
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))
ok = bcrypt.checkpw(password.encode(), hashed)
```

Argon2id is also a good modern choice.

### Practical lab
1. Check TLS configuration of a host you own (or a public test host that permits scanning, such as `badssl.com` for test certificates):

```bash
openssl s_client -connect badssl.com:443 -tls1
nmap --script ssl-enum-ciphers -p 443 localhost
```

2. In Juice Shop, look for an exposed file containing data, such as the `/ftp/` directory, and check whether any stored passwords look like hashes. Identify the hash format (MD5 is 32 hex characters, for example).
3. Use `hashcat` or an online check on a sample hash you created yourself to see how fast MD5 cracks compared to bcrypt. Do this only on hashes you generated.

### Prevention
- Use HTTPS everywhere with TLS 1.2 or newer, plus HSTS.
- Store passwords with bcrypt, scrypt, Argon2id, or PBKDF2 with a high work factor.
- Use standard libraries for cryptography. Do not write your own.
- Keep secrets out of source code. Use a secrets manager or environment variables.
- Encrypt sensitive data at rest and classify what counts as sensitive.

### Detection
- Use `testssl.sh` or `sslyze` to review TLS configurations.
- Search code for hard-coded secrets with `gitleaks` or `trufflehog`.

### Checklist
- [ ] I can explain why MD5 is unsuitable for passwords.
- [ ] I can explain salting and why bcrypt is slow on purpose.
- [ ] I can check a server's TLS version and ciphers.

---

## A05: Injection

### Concept
Injection happens when untrusted input is sent to an interpreter (SQL database, operating system shell, LDAP, template engine, or browser) as part of a command or query. Cross-Site Scripting (XSS) is a type of injection because the browser interprets the input as code.

### Types covered in this section
- **SQL injection (SQLi):** changing a database query
- **Command injection:** changing an operating system command
- **Cross-Site Scripting (XSS):** injecting JavaScript into a page viewed by others
- **Server-Side Template Injection (SSTI):** injecting into a template engine

### SQL injection

#### Vulnerable example
```python
username = request.args["user"]
query = f"SELECT * FROM users WHERE name = '{username}'"
cursor.execute(query)
```

With input `' OR '1'='1` the query returns every row.

#### Practical lab (DVWA, Low security)
1. Open **SQL Injection** in DVWA.
2. Enter `1` to see normal output.
3. Enter `1' OR '1'='1` and observe that all users appear.
4. Find the number of columns: `1' ORDER BY 2 -- -` (increase the number until it errors).
5. Extract data: `1' UNION SELECT user, password FROM users -- -`
6. Compare the query with the source code shown in the DVWA "View Source" button.

#### Practical lab (Juice Shop)
- Search the product list for a single quote (`'`). Observe the error message.
- Log in with `' OR 1=1 --` in the email field of the login page (challenge: "Login Admin").

#### Prevention (parameterized queries)
```python
cursor.execute("SELECT * FROM users WHERE name = %s", (username,))
```

Parameters are sent separately from the SQL, so input is never treated as code. Use an ORM and never build queries by string concatenation.

### Command injection

#### Vulnerable example
```python
import os
host = request.args["host"]
os.system("ping -c 1 " + host)      # host = "8.8.8.8; cat /etc/passwd"
```

#### Practical lab (DVWA, Command Injection module)
1. Enter `127.0.0.1` to see the ping output.
2. Enter `127.0.0.1; whoami` and then `127.0.0.1 && id`.
3. Note which separators (`;`, `&&`, `|`) work.

#### Prevention
```python
import subprocess
subprocess.run(["ping", "-c", "1", host], check=True)   # list arguments, no shell
```

Also validate input against an allowlist (for example, IP address format).

### Cross-Site Scripting (XSS)

#### Types
- **Reflected:** input from the URL is echoed back in the response
- **Stored:** input is saved (for example in a comment) and shown to other users
- **DOM-based:** client-side JavaScript writes untrusted data into the page

#### Vulnerable example
```html
<p>Welcome, <?php echo $_GET['name']; ?></p>
```

Request `?name=<script>alert(1)</script>`.

#### Practical lab (DVWA, XSS Reflected and Stored)
1. In **XSS (Reflected)**, enter `<script>alert('test')</script>` in the name field.
2. In **XSS (Stored)**, add the same payload to the guestbook and reload the page.
3. Try `<img src=x onerror=alert(1)>` if the script tag is filtered.

#### Practical lab (Juice Shop)
- Use the search box with `<iframe src="javascript:alert('xss')">`.
- Complete the "DOM XSS" challenge from the scoreboard.

#### Prevention
- Encode output for the context (HTML, attribute, JavaScript, URL).
- Use frameworks with automatic escaping (React, Jinja2 with autoescape, Django templates).
- Avoid `innerHTML` with untrusted data. Use `textContent` instead.
- Add a Content Security Policy to limit inline scripts.

```javascript
// Unsafe
element.innerHTML = userInput;
// Safe
element.textContent = userInput;
```

### Server-Side Template Injection (SSTI)

#### Vulnerable example (Flask / Jinja2)
```python
return render_template_string("Hello " + request.args["name"])
```

Test with `{{7*7}}`. If the page shows `49`, the template engine evaluated your input.

#### Prevention
Pass user data as variables, not as part of the template:
```python
return render_template_string("Hello {{ name }}", name=request.args["name"])
```

### Detection
- Enter special characters (`'`, `"`, `<`, `{{`, `;`) into every input and watch the response.
- Use Burp Suite Scanner (Professional) or ZAP to automate checks in your lab.
- Review code for string concatenation near SQL, shell, or HTML.

### Checklist
- [ ] I can exploit SQL injection in DVWA and explain why it works.
- [ ] I can explain the three types of XSS.
- [ ] I can fix an injection flaw with parameterized queries and safe APIs.

---

## A06: Insecure Design

### Concept
Insecure design is a flaw in the plan, not in the code. Even perfect implementation cannot fix a design that lacks security controls. This is why threat modeling and secure design reviews matter.

### Examples
- A password reset that asks only for the user's email, with security questions that are easy to guess
- No limit on the number of coupon codes a user can apply
- A checkout that trusts the price sent by the browser
- No rate limiting on expensive operations (for example, sending SMS codes)

### Practical lab (Juice Shop)
1. Place an order and intercept the request in Burp.
2. Change the price or quantity in the request body.
3. Note whether the server recalculates the total or accepts your value.
4. Try applying the same coupon or discount multiple times.

### Prevention
- Perform threat modeling early (for example with STRIDE). Ask, "What can go wrong with this feature?"
- Use secure design patterns: least privilege, defense in depth, fail securely.
- Enforce business rules on the server. The client is never trusted.
- Add rate limits and abuse controls to sensitive flows.

### Threat modeling exercise
Take a simple login-and-checkout feature and list the threats using STRIDE:

| Letter | Threat | Example question |
|---|---|---|
| S | Spoofing | Can someone pretend to be another user? |
| T | Tampering | Can prices or roles be changed in transit? |
| R | Repudiation | Can a user deny an action with no log? |
| I | Information disclosure | Can errors leak data? |
| D | Denial of service | Can one user lock out others? |
| E | Elevation of privilege | Can a user become admin? |

### Checklist
- [ ] I can explain why insecure design differs from a coding bug.
- [ ] I can perform a basic STRIDE analysis on a feature.

---

## A07: Authentication Failures

### Concept
Weaknesses in login, session handling, and credential management allow attackers to impersonate users. This was previously named "Identification and Authentication Failures."

### How it happens
- Weak or default passwords allowed
- No protection against brute force or credential stuffing
- Session IDs in URLs or not invalidated at logout
- Missing or weak multi-factor authentication (MFA)
- Verbose error messages ("username does not exist" vs. "wrong password")
- Predictable password reset tokens

### Practical lab (Juice Shop)
1. Find the admin account email on the product reviews or about page.
2. Try a password reset flow and examine the request for predictable values.
3. Attempt a brute force with a short list in Burp Intruder, against your own lab account only:
   - Load the login request into Intruder.
   - Mark the password field as the payload position.
   - Use a small wordlist and check response length or status for differences.
4. Solve the "Login Admin" challenge with SQL injection and note that it is an authentication failure caused by injection.

### Practical lab (DVWA)
- Open **Brute Force** and compare the login behavior at Low and High security levels.
- Note the difference in protection and why.

### Prevention
- Require MFA for sensitive accounts.
- Enforce strong passwords and check against breached password lists (for example, the Have I Been Pwned Pwned Passwords API).
- Rate limit and lock out (or slow down) login attempts, returning a generic message.
- Regenerate session IDs at login and invalidate them at logout.
- Set cookie flags: `HttpOnly`, `Secure`, `SameSite`.

```python
response.set_cookie("session", value, httponly=True, secure=True, samesite="Lax")
```

- Use short-lived, random, single-use password reset tokens.

### Detection
- Test lockout and rate limits.
- Inspect cookies in DevTools and check flags.
- Check whether the session ID changes after login.

### Checklist
- [ ] I can explain why generic login error messages matter.
- [ ] I can check cookie flags in DevTools.
- [ ] I can describe three controls against brute force.

---

## A08: Software or Data Integrity Failures

### Concept
Code, updates, or data are trusted without verification. An attacker who can change what the system loads or deserializes can run their own code. This includes insecure deserialization and unsigned updates.

### How it happens
- Loading scripts or libraries from a CDN without integrity checks
- Auto-updates without signature verification
- Deserializing untrusted data with unsafe formats (Python `pickle`, Java serialization, PHP `unserialize`)
- CI/CD pipelines that trust unverified inputs

### Vulnerable example (Python pickle)
```python
import pickle
data = request.get_data()
obj = pickle.loads(data)     # attacker-controlled bytes can execute code
```

### Secure example
```python
import json
obj = json.loads(request.get_data())   # data-only format, no code execution
```

### Practical lab
1. Create a harmless proof-of-concept in your lab only:

```python
import pickle, os
class Demo:
    def __reduce__(self):
        return (os.system, ("echo pickle code ran",))
payload = pickle.dumps(Demo())
pickle.loads(payload)   # prints "pickle code ran"
```

2. Explain why `json.loads` cannot do this.
3. Add Subresource Integrity to a script tag in an HTML page you create:

```html
<script src="https://cdn.example.com/lib.js"
        integrity="sha384-BASE64HASH"
        crossorigin="anonymous"></script>
```

Generate the hash with: `openssl dgst -sha384 -binary lib.js | openssl base64 -A`

### Prevention
- Verify digital signatures on updates and artifacts.
- Use safe data formats (JSON) and avoid deserializing untrusted objects.
- Use Subresource Integrity (SRI) for third-party scripts.
- Protect CI/CD pipelines: restricted write access, signed commits, and reviewed pipeline changes.

### Checklist
- [ ] I can explain why deserializing untrusted data is dangerous.
- [ ] I can add SRI to a script tag.

---

## A09: Security Logging and Alerting Failures

### Concept
If security events are not logged, or logs are not monitored, attacks go unnoticed. This category covers both what is logged and whether anyone acts on it.

### What should be logged
- Login successes and failures
- Access control failures
- Input validation failures
- Changes to privileges and sensitive data
- Use of admin functions
- Errors and exceptions (without secrets)

### What should not be logged
- Passwords, session tokens, card numbers, or full personal data

### Practical lab (Juice Shop)
1. Log in with a wrong password several times.
2. Check the container output:

```bash
docker logs juice-shop --tail 50
```

3. Identify which events are logged and which are missing.
4. Write a short note listing the events you would expect a secure application to record.

### Practical lab (Linux host, linking to Level 6 notes)
```bash
sudo grep "Failed password" /var/log/auth.log | tail
journalctl -u ssh --since "1 hour ago"
```

### Prevention
- Log security-relevant events in a structured format (JSON) with timestamp, user, source IP, and outcome.
- Protect logs from tampering and forward them to a central system.
- Set alerts for patterns such as many failed logins from one IP.
- Create and test an incident response plan.

Example structured log entry:
```python
logger.warning("login_failed", extra={"user": username, "ip": request.remote_addr})
```

### Detection
- Test whether failed logins and access denials generate entries.
- Trigger a test alert and confirm someone receives it.

### Checklist
- [ ] I can list events that must be logged and what must never be logged.
- [ ] I can find failed login events in logs on a Linux host.

---

## A10: Mishandling of Exceptional Conditions

### Concept
Applications fail in unsafe ways when errors occur. They may reveal internal details, skip security checks, or leave the system in an inconsistent state. This category was added in 2025.

### How it happens
- Stack traces or database errors shown to users
- Fail-open logic: an error in an authorization check grants access
- Unhandled exceptions that skip cleanup (leaving locks, sessions, or temp files)
- Race conditions when error handling is not atomic

### Vulnerable example
```python
def is_admin(user):
    try:
        return check_role_service(user) == "admin"
    except Exception:
        return True     # fail open: any error grants admin
```

### Secure example
```python
def is_admin(user):
    try:
        return check_role_service(user) == "admin"
    except Exception:
        logger.exception("role check failed")
        return False    # fail closed
```

### Practical lab (Juice Shop)
1. Trigger errors by sending malformed input, such as a very long string or an invalid JSON body in Burp Repeater.
2. Look at the response. Does it show stack traces, file paths, or SQL?
3. Compare with how the same input is handled in a normal request.

### Practical lab (DVWA)
- Enter a single quote into an input field and note whether the error message reveals the database type or query.

### Prevention
- Fail closed for security decisions.
- Show users generic error messages and log details server-side.
- Use `try/finally` or context managers to guarantee cleanup.
- Test error paths as carefully as normal paths.

### Detection
- Fuzz inputs with Burp Intruder or `ffuf` against your lab.
- Review code for `except: pass` and broad exception handling around security checks.

### Checklist
- [ ] I can explain fail-open vs. fail-closed.
- [ ] I can identify an error response that leaks internal details.

---

## Practice plan

| Week | Focus | Labs |
|---|---|---|
| 1 | Setup and reconnaissance | Lab setup, Burp basics, A02, A09 |
| 2 | Access control and authentication | A01, A07 |
| 3 | Injection | A05 (SQLi, XSS, command injection) |
| 4 | Cryptography, integrity, and supply chain | A03, A04, A08 |
| 5 | Design and error handling | A06, A10 |
| 6 | Full review | Juice Shop challenges by difficulty, DVWA at High security |

Work through each category until you can complete the lab without the notes, then move to the next.

---

## Recommended free resources

- OWASP Top 10: owasp.org/Top10
- OWASP Web Security Testing Guide (WSTG): owasp.org/www-project-web-security-testing-guide
- PortSwigger Web Security Academy (free labs, very high quality): portswigger.net/web-security
- TryHackMe and Hack The Box for guided and challenge-based practice
- OWASP Juice Shop challenges and the official pwning guide

---

## Ethics and safety reminder

- Test only systems you own or have written permission to test.
- Keep your labs on a private network or local machine.
- Never use the techniques in this document against real users, companies, or public services without authorization.
