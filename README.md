# PhishGuard — Phishing Triage & Awareness Toolkit

**DecodeLabs Cyber Security Project 3 — Phishing Awareness Analysis**

PhishGuard is a defensive, educational web application that helps non-expert users and cybersecurity students triage suspicious emails and messages. It runs a deterministic, rule-based detection engine entirely in the browser — no message is ever sent, no credential is ever collected, and no submitted URL is ever automatically opened.

> ⚠️ **Educational / defensive tool only.** PhishGuard does not send phishing emails, harvest credentials, execute attachments, scan third-party systems, or impersonate real individuals. All simulation content uses fictional organizations, people, and domains.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Problem Statement](#problem-statement)
3. [Objectives](#objectives)
4. [Features](#features)
5. [Architecture](#architecture)
6. [Technology Stack](#technology-stack)
7. [Installation](#installation)
8. [Environment Variables](#environment-variables)
9. [Running the Application](#running-the-application)
10. [Data Storage](#data-storage)
11. [Testing](#testing)
12. [Example Analysis](#example-analysis)
13. [Screenshots](#screenshots)
14. [Security Considerations](#security-considerations)
15. [Ethical Considerations](#ethical-considerations)
16. [Limitations](#limitations)
17. [Future Improvements](#future-improvements)
18. [Skills Demonstrated](#skills-demonstrated)
19. [License](#license)

---

## Project Overview

Phishing remains one of the most common initial-access techniques in real-world attacks, and most breaches begin with a message a human was tricked into trusting. PhishGuard was built to demonstrate **practical threat-triage skills** rather than phishing theory alone: given a suspicious message, it independently analyzes sender identity, headers, URLs, and language patterns, and produces an explainable risk classification with a recommended action.

The project is based on the DecodeLabs Cyber Security Industrial Training Kit — Project 3: *Phishing Awareness Analysis*, whose stated goal is:

> "Analyze sample emails or messages to identify phishing attempts."

PhishGuard implements that goal as a working application rather than a theoretical writeup.

## Problem Statement

Employees and everyday users are the primary target of phishing, spear phishing, whaling, smishing, vishing, quishing, and callback-scam ("TOAD") campaigns. Most people:

- Cannot reliably read a domain to find its true registrable root.
- Do not recognize social-engineering triggers (urgency, authority, curiosity, fear, greed) as they're being used against them.
- Have no simple way to check sender/header consistency before acting.
- Rarely get realistic, low-stakes practice at spotting these patterns.

There is a need for an accessible, transparent tool that **explains its reasoning** rather than issuing an opaque verdict, and that lets people practice recognizing these patterns safely.

## Objectives

- Analyze suspicious emails/messages using deterministic, explainable rules.
- Detect suspicious links, domains, subdomains, typosquatting, homograph and combosquatting attacks.
- Analyze sender information, including From / Reply-To / Return-Path consistency.
- Identify cognitive/social-engineering triggers: urgency, authority, curiosity, fear, greed.
- Identify MFA fatigue, callback scams, quishing, deepfake voice/video social engineering, browser-in-the-browser deception, and dangerous attachments.
- Classify each message as **SAFE**, **SUSPICIOUS**, or **MALICIOUS**, with a numeric score and a plain-language explanation.
- Provide a "Pause, Verify, Report" workflow and an interactive triage decision tree.
- Provide safe, fictional phishing simulations for awareness training with a non-punitive awareness score.
- Generate and store analysis reports locally, with export/print/copy support.

## Features

### Core analysis
- **Phishing Analyzer** — paste sender, From/Reply-To/Return-Path, subject, body, extra URLs, and raw headers; get a full threat assessment.
- **Rule-based detection engine** — ~20 independent detectors covering urgency, authority, curiosity, fear, greed, credential requests, sensitive-information requests, MFA fatigue, payment requests, callback scams, bypass/secrecy requests, fake-forward chains, browser-in-the-browser language, deepfake voice/video indicators, dangerous attachment extensions, and spear-phishing context clues.
- **Sender & header analysis** — display-name/domain mismatch detection, Reply-To/Return-Path routing-mismatch detection, and raw email header parsing (From, To, Reply-To, Return-Path, Received, Authentication-Results, SPF, DKIM, DMARC, Message-ID, Date).
- **URL Analyzer** — breaks a URL into scheme, host, port, path, query, subdomain chain, and root domain; flags IP-based URLs, `@`-symbol tricks, punycode/homograph characters, excessive subdomains, URL shorteners, combosquatting, and typosquatting (via edit-distance matching against known brand names, including hyphenated tokens).
- **Configurable scoring engine** — every indicator has an editable weight (Settings page); score maps to SAFE (0–24) / SUSPICIOUS (25–74) / MALICIOUS (75–100).
- **Explainable evidence** — every finding includes the indicator, severity, matched evidence text, a plain-language explanation, and a recommended action. Never a bare "looks dangerous."

### Awareness & training
- **Red Flag Checklist** — an 11-point non-technical checklist with a live recommendation.
- **Triage Decision Tree** — interactive, step-through version of the same logic the engine applies.
- **Simulation Toolkit** — 15 fictional, clearly-labeled "EDUCATIONAL SIMULATION" scenarios across Mass Phishing, Financial & Executive Threats, Modern Vectors (callback, quishing, MFA fatigue, deepfake), and Routine Internal Impersonation. Users classify each message, then see the correct answer, red flags, and a lesson — no punitive scoring.
- **Awareness Score** — accumulates points from simulations and maps to Beginner / Developing / Practiced / Advanced (an educational indicator, not a certification).
- **Learning Center** — reference cards for phishing, spear phishing, whaling, smishing, vishing, quishing, search-engine phishing, typosquatting, homograph attacks, combosquatting, BEC, MFA fatigue, callback phishing, deepfake social engineering, header analysis, URL analysis, and social-engineering triggers.

### Reporting & history
- **Dashboard** — totals, SAFE/SUSPICIOUS/MALICIOUS counts, most common red flags, awareness score, and a recent-analyses table.
- **Reports** — full per-message threat assessment with Pause/Verify/Report guidance; print, export as JSON, or copy as text.
- **Analysis History** — searchable, deletable local history of every analysis performed.

## Architecture

PhishGuard is implemented as a **single self-contained client-side application** (one HTML file with embedded CSS and JavaScript). This is a deliberate simplification of the reference project's suggested React/Node/Express/SQLite stack: because every requirement (deterministic rule engine, URL/header parsing, simulations, local persistence, reporting) can run entirely client-side without a server, an unnecessary backend was avoided in favor of a zero-install, fully portable artifact.

```
Input (message / headers / URLs)
        ↓
 Local rule engine  ──────────────►  Text detectors (urgency, authority,
        ↓                            credential/payment/sensitive-info
 URL & domain analysis                requests, MFA fatigue, callback,
        ↓                            bypass, deepfake, attachments, …)
 Header parsing (SPF/DKIM/DMARC)
        ↓
 Scoring engine (configurable weights)
        ↓
 Classification: SAFE / SUSPICIOUS / MALICIOUS
        ↓
 Report (evidence + explanation + recommendation + Pause/Verify/Report)
        ↓
 Local storage (history, awareness score, checklist state, weights)
```

If you need the modular `frontend/`, `backend/`, `shared/` folder structure described in the original brief (e.g. to demonstrate separated concerns for a course rubric), the same functions map directly:

| Original module | Where it lives in PhishGuard |
|---|---|
| `services/phishingEngine` | Text detector functions (`detectUrgency`, `detectAuthority`, etc.) |
| `services/urlAnalyzer` | `analyzeSingleUrl`, `extractUrls`, typosquat/homograph/combosquat logic |
| `services/headerAnalyzer` | `parseHeaders`, `authVerdict` |
| `services/simulationEngine` | `SIMULATIONS` data + simulation modal/scoring logic |
| `rules/` | Weight configuration (`DEFAULT_WEIGHTS`, `getWeights`/`setWeights`) |
| `models/` (DB) | `localStorage`-backed history, awareness score, checklist state |

## Technology Stack

- **Frontend:** Vanilla HTML5, CSS3 (custom properties, no framework), and JavaScript (ES6+). No build step, no external JS framework dependency.
- **Fonts:** Inter (UI) and IBM Plex Mono (data/technical text), loaded from Google Fonts.
- **Persistence:** Browser `localStorage` (per-device, per-browser — nothing is transmitted).
- **Backend:** None required. The entire rule engine runs client-side.

This keeps the project genuinely runnable by opening a single file, while still exercising the same engineering skills (modular rule design, parsing, scoring, state management, UI/UX) called for in the brief.

## Installation

No installation, build step, or package manager is required.

```bash
git clone https://github.com/<your-username>/phishguard.git
cd phishguard
```

## Environment Variables

None. PhishGuard makes no network calls and requires no API keys. (If you extend it with the optional AI-assistance layer described in [Future Improvements](#future-improvements), any API key must be kept server-side/in an environment variable and never embedded in client-side code.)

## Running the Application

Open `index.html` (or `phishguard.html`) directly in any modern browser:

```bash
# macOS
open index.html

# Windows
start index.html

# Linux
xdg-open index.html
```

Or serve it locally (optional, useful for testing on other devices on your network):

```bash
python3 -m http.server 8080
# then visit http://localhost:8080
```

Or enable **GitHub Pages** on this repository (Settings → Pages → deploy from `main` / root) to get a live, shareable URL with zero server management.

## Data Storage

All data is stored in the browser's `localStorage`, scoped to the origin serving the file:

| Key | Contents |
|---|---|
| `pg_history` | Saved analysis reports (up to the 300 most recent) |
| `pg_awareness` | Simulation points, attempts, and correct-answer count |
| `pg_weights` | Custom scoring weights, if changed from defaults |
| `pg_checklist_state` | Checked/unchecked state of the Red Flag Checklist |

Nothing is sent to a server. Clearing browser storage (or using the in-app "Reset All Local Data" button in Settings) permanently erases this data. No real credentials, passwords, or sensitive personal information are ever stored — the tool only stores the metadata of analyses you choose to run.

## Testing

Because there is no build/bundler step, the rule engine was validated with targeted Node.js smoke tests exercising the pure detection/scoring functions (extracted from the page's script blocks) against representative cases, including:

- A safe, benign message → score 0, classification SAFE.
- The built-in demo phishing example (urgency + authority + credential request + domain mismatch + deceptive subdomain) → high score, classification MALICIOUS.
- A Business-Email-Compromise-style message (urgency + payment request + secrecy/bypass language) → classification SUSPICIOUS.
- A typosquatted domain (`amaz0n-orders.com`) → correctly flagged via edit-distance matching against the `amazon` brand token.
- Reply-To/Return-Path mismatches, IP-based URLs, punycode hostnames, and excessive-subdomain URLs.

If you add a `tests/` folder with a JS test runner (e.g. Vitest or plain Node `assert`), the exported logic under `<script>` can be lifted into standalone modules with minimal changes, since the detectors are pure functions of input text/URLs.

## Example Analysis

**Input** (built-in "Load Demo Example"):

```
Sender:     Microsoft Security
From:       security@example-security-alert.com
Reply-To:   support@example-security-alert.com
Return-Path: bounce@mail-relay.example.net
Subject:    URGENT: Your account will be locked in 30 minutes
Body:       Your account has been selected for immediate security
            verification. Failure to verify will result in account
            suspension. Click here to verify:
            https://login.example-secure-check.com.verify-user.example.net/session
```

**Output:**

- **Classification:** MALICIOUS (score ≈ 77/100)
- **Detected indicators:** Urgency, Authority Impersonation, Credential/Login Verification Request, Sender Display-Name/Domain Mismatch, Return-Path Mismatch, Excessive/Deceptive Subdomains, Suspicious Keywords in URL
- **Explanation excerpt:** *"Suspicious because the message creates artificial urgency, requests account verification, and contains a domain that does not match the claimed organization."*
- **Recommended action:** Warn User / Block & Escalate — Pause, verify through an independently obtained contact, and report through your organization's reporting mechanism.

## Screenshots

Suggested screenshots to capture for a submission or portfolio (open each page and use your browser's screenshot tool):

1. Dashboard (`#dashboard`)
2. Analyzer input form (`#analyzer`)
3. A completed phishing analysis result with evidence panels
4. URL Analyzer breakdown of a deceptive-subdomain URL
5. Header Analyzer with SPF/DKIM/DMARC results
6. Triage Decision Tree mid-flow
7. Simulation library (`#simulations`)
8. A completed simulation with feedback and lesson
9. A generated report modal
10. Learning Center card detail
11. Analysis History table

## Security Considerations

- All input is treated as plain text; nothing is evaluated, executed, or rendered as HTML from user input (output is escaped before insertion into the DOM).
- Submitted URLs are parsed as strings only — **PhishGuard never fetches, navigates to, or renders a submitted URL.**
- No attachments are ever executed; the tool only pattern-matches filenames/extensions mentioned in text.
- No external network requests are made by the application itself.
- No secrets, API keys, or credentials are stored, transmitted, or hardcoded anywhere in the codebase.
- All persisted data stays in the browser's local storage on the user's own device.

## Ethical Considerations

This project follows the source training material's ethical simulation-design principles:

- **Never:** sends real phishing messages, harvests credentials, stores passwords, executes malicious code, automatically opens suspicious URLs, attacks real domains/third-party systems, or impersonates real, identifiable individuals.
- **Always:** uses fictional organizations, people, and domains in every example and simulation; clearly labels every simulated message as an **EDUCATIONAL SIMULATION**; avoids extreme emotional distress (no death threats, family-emergency framing, or panic-inducing content) in training scenarios; treats simulation results as a learning opportunity rather than a punitive score.
- **Avoids false certainty:** the tool never states a message is "definitely safe" — only that it is "low-risk based on the indicators analyzed," consistent with its role as a triage aid rather than a replacement for human judgment.

## Limitations

- Classification relies on **deterministic keyword and structural heuristics**, not machine learning or live threat intelligence — it will miss novel phrasing and can occasionally over- or under-flag based on wording alone.
- Typosquatting/combosquatting/homograph detection use simplified heuristics (edit distance, a fixed brand list, non-ASCII/punycode checks) rather than a full public-suffix-list-aware WHOIS/DNS lookup.
- QR-code (quishing) analysis is covered as a text-based/educational scenario; the tool does not decode actual QR image files.
- No live integrations (VirusTotal, Google Safe Browsing, WHOIS, DNS, SIEM) are included by design — see below.
- This is a **triage assistant for awareness and practice**, not a certified security product, and should not be the sole basis for real incident-response decisions.

## Future Improvements

All of the following are optional, deliberately excluded from the base project so it works with zero external dependencies:

- Threat-intelligence lookups: VirusTotal / Google Safe Browsing URL reputation, domain reputation, WHOIS, and DNS analysis.
- SIEM integration and automatic SOC ticket creation.
- Enterprise mail integration (Microsoft 365 / Google Workspace) for direct message ingestion.
- Advanced ML-based classification layered on top of (not replacing) the deterministic rule engine.
- Optional AI-assisted explanation layer: architecture would be `Input → local rule engine → URL/header analysis → optional AI explanation (advisory only) → combined report`, with the message *"AI explanation unavailable — deterministic analysis completed"* shown whenever no API key is configured, and all keys kept server-side.
- Native QR image decoding (client-side, e.g. via a `jsQR`-style library) feeding into the existing URL analyzer, with results still never auto-opened.

## Skills Demonstrated

Threat analysis · social-engineering awareness · email/message analysis · URL and domain analysis · email header analysis · rule-based phishing detection · security awareness training design · triage workflow design · defensive security engineering.

## License

Educational project. Use, adapt, and extend freely for learning and portfolio purposes. All example organizations, domains, and identities throughout the application are fictional.
