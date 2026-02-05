# Lab 2

## Task 1

### Screenshots of generated diagrams

![alt text](lab2/baseline/data-asset-diagram.png)

![alt text](lab2/baseline/data-flow-diagram.png)


### Top 5 Risks table

| Severity | Category                  | Asset        | Likelihood | Impact | Score |
|----------|---------------------------|--------------|------------|--------|-------|
| elevated | unencrypted-communication | user-browser | likely     | high   | 433   |
| elevated | missing-authentication    | juice-shop   | likely     | medium | 432   |
| elevated | cross-site-scripting      | juice-shop   | likely     | medium | 432   |
| elevated | unencrypted-communication | reverse-proxy| likely     | medium | 432   |
| medium   | cross-site-request-forgery | juice-shop   | very-likely| low    | 241   |

Score in the table calculated by rules:
- Severity: critical (5) > elevated (4) > high (3) > medium (2) > low (1)
- Likelihood: very-likely (4) > likely (3) > possible (2) > unlikely (1)
- Impact: high (3) > medium (2) > low (1)

### Analysis of critical security concerns identified

1. Unencrypted-communication: Data transfer between the user's browser and the application takes place over HTTP without encryption
2. Missing-authentication: Juice Shop has insufficient authentication for some functions
3. Cross-site-scripting: Application is vulnerable to XSS attacks
4. Cross-site-request-forgery: Juice Shop is vulnerable to CSRF attacks

## Task 2

### Screenshots of generated diagrams

![alt text](lab2/secure/data-asset-diagram.png)

![alt text](lab2/secure/data-flow-diagram.png)

### Risk Category Delta Table
| Category                          | Baseline | Secure | Δ   |
|------------------------------------|---------:|-------:|----:|
| container-baseimage-backdooring    |        1 |      1 |   0 |
| cross-site-request-forgery         |        2 |      2 |   0 |
| cross-site-scripting               |        1 |      1 |   0 |
| missing-authentication             |        1 |      1 |   0 |
| missing-authentication-second-factor |      2 |      2 |   0 |
| missing-build-infrastructure       |        1 |      1 |   0 |
| missing-hardening                  |        2 |      2 |   0 |
| missing-identity-store             |        1 |      1 |   0 |
| missing-vault                      |        1 |      1 |   0 |
| missing-waf                        |        1 |      1 |   0 |
| server-side-request-forgery        |        2 |      2 |   0 |
| unencrypted-asset                  |        2 |      1 |  -1 |
| unencrypted-communication          |        2 |      0 |  -2 |
| unnecessary-data-transfer          |        2 |      2 |   0 |
| unnecessary-technical-asset        |        2 |      2 |   0 |

### Delta Run

- **Change Made**: Implemented HTTPS encryption for communication links between User Browser, Reverse Proxy, and Juice Shop Application.

- **Observed Result**: Reduced unencrypted-communication risks from 2 to 0 (Δ = -2), while maintaining other risk categories at baseline levels.

- **Why**: HTTPS encryption protects sensitive authentication data and improves the confidentiality of user credentials and session tokens.