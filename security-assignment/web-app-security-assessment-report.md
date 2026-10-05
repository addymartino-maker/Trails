# Web Application Security Assessment Report

> **Template + worked example.** Replace the bracketed `[...]` placeholders with your
> own details. The findings in Section 6 are **real results from OWASP Juice Shop**, a
> deliberately vulnerable application tested in an authorized sandbox — use them as a
> model for how to document each finding, then add/adjust for your own target.

---

## 1. Document Control

| Field | Value |
|---|---|
| Report title | Web Application Security Assessment — [Target Name] |
| Author | [Your Name], [Course / Student ID] |
| Date | [YYYY-MM-DD] |
| Version | 1.0 |
| Classification | Confidential — Coursework |

---

## 2. Authorization & Scope

> **This section is not optional — it is the single most important part of any real
> assessment.** Never test a system without written authorization from its owner.

- **Target(s):** [e.g., `http://localhost:3000` — OWASP Juice Shop, local Docker instance]
- **Authorization basis:** [e.g., "Self-hosted deliberately vulnerable application; I own
  the instance and the host." OR "Signed engagement / scope document dated [...]"]
- **In scope:** [URLs, hostnames, IP ranges, specific functionality]
- **Out of scope:** [Anything you are NOT permitted to touch — DoS, social engineering,
  physical access, third-party services, etc.]
- **Testing window:** [Start] – [End]
- **Rules of engagement:** [Rate limits, no destructive actions, data handling, contact
  for emergencies]

---

## 3. Executive Summary

A security assessment of [Target] was performed between [dates] following the
OWASP Web Security Testing Guide (WSTG) and evaluating against the OWASP Top 10 (2021).

The assessment identified **[N]** findings: **[x] Critical, [x] High, [x] Medium,
[x] Low, [x] Informational**. The most significant issues were [1–2 sentence summary of
the worst problems and their business impact].

Overall risk rating: **[Critical / High / Medium / Low]**.

---

## 4. Methodology

Testing followed a structured black-box / grey-box methodology:

1. **Reconnaissance & mapping** — enumerate pages, endpoints, parameters, technologies.
2. **Configuration & deployment testing** — headers, TLS, error handling, exposed files.
3. **Authentication testing** — login logic, password policy, brute-force protection.
4. **Session management** — token generation, storage, expiry, logout.
5. **Authorization testing** — access control, IDOR, privilege escalation.
6. **Input validation** — injection (SQL/NoSQL/command), XSS, SSTI.
7. **Business logic** — workflow abuse, race conditions.
8. **Client-side testing** — DOM XSS, sensitive data in the browser.

**Tools used:** [browser dev tools, Burp Suite / OWASP ZAP, nmap, nikto, sqlmap, curl, etc.]

Severity is rated using **CVSS v3.1** base scores.

---

## 5. Findings Summary

| # | Finding | OWASP Top 10 | Severity | CVSS |
|---|---|---|---|---|
| F-01 | [Title] | A01: Broken Access Control | High | 7.5 |
| F-02 | [Title] | A03: Injection | Critical | 9.1 |
| ... | | | | |

---

## 6. Detailed Findings

> Repeat this block for each finding.

### F-01 — [Finding Title]

- **Severity:** [Critical / High / Medium / Low]  |  **CVSS v3.1:** [score] ([vector])
- **OWASP category:** [e.g., A01:2021 Broken Access Control]
- **Affected component:** [URL / parameter / endpoint]

**Description**
[What the vulnerability is, in plain language.]

**Evidence / Steps to Reproduce**
1. [Step]
2. [Step]
3. [Observed result — include request/response snippets or screenshots]

```
[paste HTTP request/response or payload here]
```

**Impact**
[What an attacker could achieve — data theft, account takeover, etc.]

**Remediation**
[Specific, actionable fix — parameterized queries, access-control checks, headers, etc.]

**References**
- [OWASP WSTG / CWE / vendor docs]

---

## 7. Remediation Roadmap

| Priority | Finding(s) | Recommended action | Effort |
|---|---|---|---|
| 1 | F-02 | [Fix] | [S/M/L] |
| 2 | F-01 | [Fix] | [S/M/L] |

---

## 8. Appendix

- **A. Tool output** — [raw scanner logs]
- **B. Testing checklist** — OWASP WSTG coverage
- **C. Glossary** — CVSS, IDOR, XSS, etc.
