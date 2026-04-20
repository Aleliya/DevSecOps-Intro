# Lab 11

## Task 1

### Why reverse proxies are valuable for security

The reverse proxy handles SSL/TLS encryption, offloading the main application from resource-intensive encryption/decryption operations. The proxy can automatically add important security headers (HSTS, CSP, X-Frame Options, etc.) to all responses, has the ability to block malicious requests, DDoS attacks, SQL injections at the proxy level until reaching the main application, and simplifies access control, monitoring, and logging of all incoming traffic.

### Why hiding direct app ports reduces attack surface

The direct ports of the application are not visible from the outside, which eliminates the possibility of direct attacks on vulnerabilities in the application itself. All requests pass through a secure proxy layer with additional validation, reducing the number of open ports on the host, which reduces the overall attack surface and makes it possible to hide the real version and technology stack of the backend application.

### ```docker compose ps``` output showing only Nginx has published host ports

```bash
NAME            IMAGE                           COMMAND                  SERVICE   CREATED          STATUS          PORTS
lab11-juice-1   bkimminich/juice-shop:v19.0.0   "/nodejs/bin/node /j…"   juice     11 seconds ago   Up 10 seconds   3000/tcp
lab11-nginx-1   nginx:stable-alpine             "/docker-entrypoint.…"   nginx     10 seconds ago   Up 9 seconds    0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp, 0.0.0.0:8443->8443/tcp, [::]:8443->8443/tcp
```


## Task 2

### `headers-https.txt`

```bash
HTTP/2 200 
server: nginx
date: Sun, 19 Apr 2026 13:53:36 GMT
content-type: text/html; charset=UTF-8
content-length: 75002
feature-policy: payment 'self'
x-recruiting: /#/jobs
accept-ranges: bytes
cache-control: public, max-age=0
last-modified: Sun, 19 Apr 2026 13:53:25 GMT
etag: W/"124fa-19aa6b0f2e1"
vary: Accept-Encoding
strict-transport-security: max-age=31536000; includeSubDomains; preload
x-frame-options: DENY
x-content-type-options: nosniff
referrer-policy: strict-origin-when-cross-origin
permissions-policy: camera=(), geolocation=(), microphone=()
cross-origin-opener-policy: same-origin
cross-origin-resource-policy: same-origin
content-security-policy-report-only: default-src 'self'; img-src 'self' data:; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'
```

### Security Header Analysis

- **X-Frame-Options: DENY**: Prevents clickjacking attacks by denying the page from being embedded in frames/iframes
- **X-Content-Type-Options: nosniff**: Prevents MIME type sniffing, protecting against certain XSS attacks
- **Strict-Transport-Security (HSTS)**: Forces browsers to use HTTPS only, protects against protocol downgrade attacks
- **Referrer-Policy: strict-origin-when-cross-origin**: Controls referrer information sent with requests for privacy and security
- **Permissions-Policy: camera=(), geolocation=(), microphone=()**: Restricts access to sensitive browser APIs
- **COOP/CORP (Cross-Origin-Opener-Policy/Cross-Origin-Resource-Policy)**: Controls cross-origin resource sharing and opener policies
- **CSP-Report-Only**: Content Security Policy in report-only mode to monitor potential violations without blocking

## Task 3

### TLS/testssl summary

1. Summarize TLS protocol support from testssl scan:

    - SSLv2: not offered (OK)
    - SSLv3: not offered (OK)
    - TLS 1.0: not offered
    - TLS 1.1: not offered
    - TLS 1.2: offered (OK)
    - TLS 1.3: offered (OK)

2. List cipher suites that are supported:

    - TLSv1.2: ECDHE-RSA-AES256-GCM-SHA384, ECDHE-RSA-AES128-GCM-SHA256
    - TLSv1.3: TLS_AES_256_GCM_SHA384, TLS_CHACHA20_POLY1305_SHA256, TLS_AES_128_GCM_SHA256

3. Why TLSv1.2+ is required:

    - TLS 1.0/1.1 have known security vulnerabilities (BEAST, POODLE, etc.)
    - TLS 1.3 provides the strongest security with forward secrecy and modern cipher suites
    - Older protocols can be exploited for man-in-the-middle attacks

4. Warnings or vulnerabilities from testssl output:

    - Chain of trust: NOT ok (self-signed) - expected for local development
    - OCSP/CRL/CT/CAA: Not offered - expected without public CA certificate
    - OCSP stapling: not offered - disabled in nginx.conf for self-signed cert

5. Confirm HSTS header appears only on HTTPS responses (not HTTP):

    HSTS header appears only on HTTPS responses: `strict-transport-security: max-age=31536000; includeSubDomains; preload`

### Rate limiting & timeouts

1. Rate-limit test output:

    ```text
        401
        401
        401
        401
        401
        401
        429
        429
        429
        429
        429
        429
    ```

2. Rate limit configuration

    - **Configuration**: `rate=10r/m`, `burst=5`
    - **Behavior**: First 6 requests return 401 (invalid credentials), next 6 return 429 (rate limited)
    - **Balance**: 10 requests/minute with burst allowance of 5 provides good security vs usability balance - prevents brute force attacks while allowing legitimate login attempts

3. Timeout settings in ```nginx.conf```:

    - `client_body_timeout 10s`: Prevents slowloris attacks by limiting time to send request body
    - `client_header_timeout 10s`: Prevents slowloris by limiting header transmission time
    - `proxy_read_timeout 30s`: Reasonable timeout for upstream response reading
    - `proxy_send_timeout 30s`: Reasonable timeout for sending data to upstream
    - `keepalive_timeout 10s`: Balances connection reuse with resource management

    Trade-offs:

    - Shorter timeouts improve security against DoS but may cause issues with slow clients/networks
    - Longer timeouts improve compatibility but increase resource consumption
    - Current settings provide good balance for this application scenario

4. Relevant lines from ```access.log``` showing 429 responses:

    ```bash
    172.19.0.1 - - [19/Apr/2026:13:54:50 +0000] "POST /rest/user/login HTTP/2.0" 429 162 "-" "curl/8.5.0" rt=0.000 uct=- urt=-
    172.19.0.1 - - [19/Apr/2026:13:54:50 +0000] "POST /rest/user/login HTTP/2.0" 429 162 "-" "curl/8.5.0" rt=0.000 uct=- urt=-
    172.19.0.1 - - [19/Apr/2026:13:54:50 +0000] "POST /rest/user/login HTTP/2.0" 429 162 "-" "curl/8.5.0" rt=0.000 uct=- urt=-
    172.19.0.1 - - [19/Apr/2026:13:54:50 +0000] "POST /rest/user/login HTTP/2.0" 429 162 "-" "curl/8.5.0" rt=0.000 uct=- urt=-
    172.19.0.1 - - [19/Apr/2026:13:54:50 +0000] "POST /rest/user/login HTTP/2.0" 429 162 "-" "curl/8.5.0" rt=0.000 uct=- urt=-
    172.19.0.1 - - [19/Apr/2026:13:54:50 +0000] "POST /rest/user/login HTTP/2.0" 429 162 "-" "curl/8.5.0" rt=0.000 uct=- urt=-
    ```