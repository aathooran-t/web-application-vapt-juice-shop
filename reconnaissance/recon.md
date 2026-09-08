 Reconnaissance

## 1. Objective

The objective of the reconnaissance phase was to perform initial information gathering against the OWASP Juice Shop web application running in a controlled local Docker environment.

The reconnaissance phase focused on identifying:

* The target application and environment
* Open ports and services
* Application technologies
* HTTP response headers
* Discoverable directories and endpoints
* Information exposed through `robots.txt`

No vulnerability was declared solely from reconnaissance observations. Potential findings will be manually validated during later phases.

---

## 2. Target Information

| Item             | Details                 |
| ---------------- | ----------------------- |
| Target           | OWASP Juice Shop        |
| Target URL       | `http://127.0.0.1:3000` |
| Environment      | Local Docker lab        |
| Operating System | Kali Linux              |
| Target Port      | TCP/3000                |
| Assessment Type  | Web Application VAPT    |

---

## 3. Environment Verification

The local assessment environment was verified before beginning reconnaissance.

### Kali Linux

The current user was verified using:

```bash
whoami
```

Result:

```text
kali
```

The local IP address was identified using:

```bash
ip addr
```

The Kali Linux system was assigned:

```text
10.0.3.15/24
```

### Docker

Docker was verified using:

```bash
docker --version
```

Result:

```text
Docker version 28.5.2+dfsg4
```

The Docker service was confirmed to be running.

---

## 4. Target Identification

OWASP Juice Shop was running locally through Docker and was accessible at:

```text
http://127.0.0.1:3000
```

The application was successfully accessed through a web browser.

---

## 5. Port and Service Enumeration

### Nmap

The following command was used to identify the service running on port 3000:

```bash
nmap -sV -p 3000 127.0.0.1
```

### Results

The scan confirmed:

* Host `127.0.0.1` was reachable.
* TCP port `3000` was open.
* The service returned an HTTP response.
* The response identified the application as OWASP Juice Shop.

The service fingerprint was not fully recognized by Nmap, but the HTTP response confirmed that a web application was running.

### Security-Relevant Observations

The HTTP response contained several headers, including:

```text
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'
X-Recruiting: /#/jobs
```

The `Access-Control-Allow-Origin: *` header was recorded as an observation for later validation. Its presence alone does not establish a CORS vulnerability.

---

## 6. Technology Fingerprinting

### WhatWeb

WhatWeb was used to identify technologies and characteristics of the web application:

```bash
whatweb http://127.0.0.1:3000
```

### Results

The scan identified:

```text
200 OK
HTML5
Script[module]
Title[OWASP Juice Shop]
X-Frame-Options[SAMEORIGIN]
X-Content-Type-Options[nosniff]
Access-Control-Allow-Origin
Feature-Policy
X-Recruiting
```

### Interpretation

The results confirmed that:

* The target is an HTML5 web application.
* JavaScript modules are used by the application.
* The application title identifies it as OWASP Juice Shop.
* Several HTTP security-related headers are present.

---

## 7. HTTP Response Header Analysis

The HTTP response headers were collected using:

```bash
curl -I http://127.0.0.1:3000
```

Important headers observed included:

```text
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'
X-Recruiting: /#/jobs
Content-Type: text/html; charset=UTF-8
```

### Observations

`X-Content-Type-Options: nosniff` helps prevent MIME-type sniffing.

`X-Frame-Options: SAMEORIGIN` restricts framing of the application to the same origin.

`Access-Control-Allow-Origin: *` allows cross-origin requests from any origin and was therefore recorded for further investigation during the vulnerability assessment phase.

The `X-Recruiting` header exposes an application route:

```text
/#/jobs
```

This was recorded as an information-disclosure observation rather than immediately classified as a vulnerability.

---

## 8. Directory and Endpoint Enumeration

Gobuster was used to identify potentially accessible directories and endpoints:

```bash
gobuster dir -u http://127.0.0.1:3000 -w /usr/share/wordlists/dirb/common.txt --exclude-length 9393
```

The exclusion of response length `9393` was required because the application returned the same response size for nonexistent paths.

### Discovered Endpoints

The following paths were identified:

| Endpoint       | Status | Observation           |
| -------------- | -----: | --------------------- |
| `/api`         |    500 | API-related endpoint  |
| `/apis`        |    500 | API-related endpoint  |
| `/assets`      |    301 | Redirect              |
| `/ftp`         |    200 | Accessible endpoint   |
| `/media`       |    301 | Redirect              |
| `/profile`     |    500 | Application endpoint  |
| `/promotion`   |    200 | Accessible endpoint   |
| `/redirect`    |    500 | Application endpoint  |
| `/rest`        |    500 | REST-related endpoint |
| `/restaurants` |    500 | Application endpoint  |
| `/restore`     |    500 | Application endpoint  |
| `/restored`    |    500 | Application endpoint  |
| `/restricted`  |    500 | Application endpoint  |
| `/robots.txt`  |    200 | Robots file           |
| `/Video`       |    200 | Accessible resource   |
| `/video`       |    200 | Accessible resource   |

The HTTP 500 responses were not automatically considered vulnerabilities because they require further investigation to determine their cause and security impact.

The `/ftp`, `/rest`, `/api`, and `/robots.txt` endpoints were considered particularly interesting for further assessment.

---


