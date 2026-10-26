
**Real-Time Threat Detection and Incident Response using a Web Application Firewall**

DeceptaGuard is an educational reverse-proxy Web Application Firewall (WAF) written in Python. It sits in front of a web application, inspects every HTTP request, scores the risk, decides whether to allow, alert on or block it, and records evidence that can be reviewed on a local dashboard.


**Authors:** Hemin Varghese and Yathin R
**Guide:** Dr. P. Shobana, Head of Department, Digital and Cyber Forensic Science
**Institution:** Sree Saraswathi Thyagaraja College (Autonomous), Pollachi
**Degree project:** B.Sc. Digital and Cyber Forensic Science, October 2026

---

## Why this project

Web applications are a common target for SQL injection, cross-site scripting (XSS), path traversal and probing. Basic filters that match a single pattern can miss encoded payloads, flag harmless input, and say little about why a request was blocked. DeceptaGuard explores a more context-aware approach: it normalises input, checks rules, watches behaviour over time, and keeps explainable evidence for each decision.

## Features

- **Reverse-proxy WAF:** all traffic passes through DeceptaGuard before reaching the application
- **Strict HTTP parsing:** malformed, ambiguous, unsupported or oversized requests are rejected instead of being guessed at
- **Input normalisation:** raw, decoded and canonical views of each value to catch simple encoding-based evasion
- **Rule-based detection:** configurable rules for SQL injection, XSS, path traversal, command injection, local file inclusion, NoSQL injection and more
- **Behavioural analysis:** request rate, path diversity, not-found ratio, failed logins, repeated findings and malformed requests per source
- **Risk scoring:** a score from 0 to 100 combining severity, confidence and context
- **Automated response:** allow, alert, block (HTTP 403) or temporary ban, depending on the mode and policy
- **Deception signals:** requests to common probe paths such as `/.env` and `/wp-login.php` raise the risk
- **Response inspection:** checks origin responses for information leakage such as stack traces, database errors and exposed secrets
- **Evidence and dashboard:** events stored in SQLite and shown on a local dashboard with attack distribution, severity, active bans and recent events
- **Three operating modes:** learning, detection and protection

## How it works

```
Client -> Ban check -> HTTP parser -> Normaliser -> Detection rules
       -> Behaviour engine -> Risk scoring -> Responder (ALLOW / ALERT / BLOCK)
       -> Origin app (OWASP Juice Shop) -> Response inspector -> Client
                                   \-> Evidence store (SQLite) -> Dashboard
```

| Component | File | Role |
|---|---|---|
| Reverse proxy | `waf.py` | Receives requests and runs the processing pipeline |
| Normaliser | `normalizer.py` | Builds normalised input views and checks for request-smuggling signs |
| Detection | `detection.py`, `rules.json` | Matches configurable rules against request fields |
| Behaviour | `behavior.py` | Correlates activity from each source over a short window |
| Scoring | `scoring.py` | Calculates the 0 to 100 risk score |
| Responder | `responder.py` | Chooses the action and manages temporary bans |
| Evidence | `evidence.py` | Stores events and findings in SQLite |
| Response inspector | `response_inspector.py` | Looks for information leakage in origin responses |
| Dashboard | `dashboard.py` | Local web interface for reviewing events |
| Tests | `tests.py`, `tests_juiceshop.py` | Benign and attack test cases |

## Tech stack

Python 3 (standard library), SQLite, JSON configuration, Docker, OWASP Juice Shop (test target), Kali Linux (attack simulation), Ubuntu, VMware Workstation.

## Lab setup

The project was tested with two virtual machines on the same network mode (both NAT or both Bridged):

- **Ubuntu VM:** runs OWASP Juice Shop in Docker, the WAF and the dashboard
- **Kali Linux VM:** sends test requests, including simulated attacks, through the WAF

## Getting started

Run this only in an isolated lab, never against systems you don't own or have permission to test.

1. Start Juice Shop in Docker on Ubuntu (it listens on `127.0.0.1:3000`):
   ```
   ./setup_juiceshop_ubuntu.sh
   ```
2. Start the WAF in detection mode:
   ```
   DG_MODE=detection python3 waf.py --target-port 3000 --listen-host 0.0.0.0 --listen-port 9090
   ```
   The `0.0.0.0` listen address lets the Kali VM reach the WAF over the lab network. Use `127.0.0.1` if you are testing on one machine.
3. Start the dashboard in a second terminal:
   ```
   python3 dashboard.py
   ```
   It runs locally on port 8001 with a development token. Do not expose it to the internet, and change the default token.
4. Run the included smoke tests:
   ```
   python3 tests_juiceshop.py
   ```
5. From Kali, send test requests to the WAF on port 9090 and watch the events appear on the dashboard.

## Test coverage

The test cases cover normal traffic plus simulated SQL injection, XSS, path traversal, command injection, local file inclusion, NoSQL injection, access to sensitive paths such as `/.env`, malformed HTTP requests, reconnaissance and GraphQL introspection. The included smoke tests confirm that benign requests pass and that the SQL injection, path traversal, honeypot path and XSS cases return the expected 403 response.


## Limitations

- It is a prototype and does not implement the full OWASP Core Rule Set
- Detection depends on the quality of the rules and the chosen mode and thresholds
- Behaviour tracking is based on source IP, which distributed attackers can evade
- No HTTP/2, HTTP/3 or TLS termination support
- Further validation of protection mode is still needed

## Future work

More detection rules, better risk-scoring, flexible response policies and notifications, dashboard filtering and report export, SIEM integration, and testing with larger and more varied traffic. Machine learning may be explored alongside the rules.

## Disclaimer

This project is for education and authorised testing only. The authors are not responsible for misuse.

## Authors

- **Hemin Varghese:** [LinkedIn](https://www.linkedin.com/in/hemin-varghese-045369437)
- **Yathin R**

## Acknowledgements

Dr. P. Shobana and the faculty of the Department of Digital and Cyber Forensic Science, Sree Saraswathi Thyagaraja College. Built with OWASP Juice Shop and guided by OWASP resources.
