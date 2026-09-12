<div align="center">

# 🛡️ CyberGuard // URL Threat Detector

**Real-time phishing URL detection using structural analysis — not stale blacklists.**

[![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-3.0-000000?logo=flask&logoColor=white)](https://flask.palletsprojects.com/)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-1.5-F7931E?logo=scikitlearn&logoColor=white)](https://scikit-learn.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Status](https://img.shields.io/badge/status-academic%20project-blue)]()

</div>

---

## Table of Contents

- [Overview](#-overview)
- [Demo](#-demo)
- [Features](#-features)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
- [Usage](#-usage)
- [Configuration](#-configuration)
- [Model Details](#-model-details)
- [Testing](#-testing)
- [Limitations](#-limitations)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)
- [License](#-license)
- [Acknowledgments](#-acknowledgments)

---

## 📌 Overview

Phishing attacks rely on deceptive URLs to steal credentials or deliver malware.
Most defenses — Google Safe Browsing, PhishTank, corporate proxies — depend on
**blacklists**: a URL has to be seen, reported, and catalogued before it gets
blocked. That's a losing race against attackers who register a domain, run a
phishing campaign for a few hours, and abandon it long before any blacklist
catches up.

**CyberGuard** takes a different approach. Instead of asking *"has this exact
link been reported before?"*, it asks *"does this link's structure look like
a phishing link?"* — using a **Random Forest classifier** trained on lexical
and structural features extracted directly from the URL string. No page is
ever opened, rendered, or downloaded, so the scan is both instant (**~12 ms**
average) and inherently safe to run on links you haven't vetted yet.

## 🎥 Demo

| Input | Safe Result | Threat Result |
|---|---|---|
| ![Input screen](screenshots/01_input.png) | ![Safe result](screenshots/02_safe_result.png) | ![Threat result](screenshots/03_threat_result.png) |

A live, in-browser version of the UI (running a lightweight client-side approximation of the model) is included in this repo as `CyberGuard_Live_Demo.html` — open it directly in any browser, no server required.

## ✨ Features

- **Zero-day capable** — flags never-before-seen phishing domains using structure alone, no blacklist required
- **Sub-50ms verdicts** — whitelist check + feature extraction + inference typically completes in ~12 ms end-to-end
- **Explainable output** — every "Threat Detected" verdict lists the specific structural red flags that triggered it
- **Trusted-domain whitelist** — well-known domains short-circuit straight to "Safe" to avoid false alarms
- **Input validation** — rejects empty or malformed input before it ever reaches the model
- **Clean REST API** — a single `POST /scan` endpoint returns JSON, easy to wire into other tools
- **Dark, glassmorphic UI** — single-page front end with animated result cards, no framework required

## 🏗️ Architecture

![System architecture](docs/architecture.png)

1. **User Interface** — a single-page front end where the user pastes a URL.
2. **Whitelist Filter** — checks the domain against a curated list of trusted, high-traffic domains; a match returns "Safe" immediately.
3. **Feature Extraction** (`feature_extraction.py`) — otherwise, five structural features are computed straight from the URL string:

   | Feature | Description |
   |---|---|
   | `url_length` | Total character length of the URL |
   | `domain_length` | Character length of the domain/host |
   | `special_chars` | Count of `.` `-` `@` characters |
   | `has_ip_address` | Whether the host is a raw IP address instead of a domain |
   | `is_https` | Whether the URL uses HTTPS |

4. **Random Forest Classifier** (`url_classifier.pkl`) — the feature vector is scored, returning a hazard confidence value.
5. **Result Renderer** — a colour-coded verdict (🟢 Safe / 🔴 Threat Detected) is displayed instantly, along with the reasoning behind it.

## 🧰 Tech Stack

| Layer | Technology |
|---|---|
| Language | Python 3.9+ |
| Web framework | Flask |
| ML / Data | scikit-learn, pandas, NumPy |
| Model persistence | joblib |
| Front end | HTML5, CSS3 (glassmorphism), vanilla JavaScript |
| Fonts | Space Grotesk, Inter, JetBrains Mono (Google Fonts) |

## 📁 Project Structure

```
cyberguard/
├── app.py                    # Flask server — routes: / and /scan
├── feature_extraction.py     # URL → structural feature vector
├── train_model.py            # Trains the Random Forest, saves url_classifier.pkl
├── generate_dataset.py       # Builds data/url_dataset.csv
├── url_classifier.pkl        # Pre-trained model (ready to use)
├── requirements.txt
├── LICENSE
├── .gitignore
├── data/
│   ├── url_dataset.csv       # 8,000 labelled training URLs
│   ├── metrics.json          # Latest training run metrics
│   └── js_model_approx.json  # Coefficients for the in-browser demo model
├── templates/
│   └── index.html
├── static/
│   ├── css/style.css         # Dark glassmorphic theme
│   └── js/script.js
├── screenshots/
│   ├── 01_input.png
│   ├── 02_safe_result.png
│   └── 03_threat_result.png
├── docs/
│   └── architecture.png
└── CyberGuard_Live_Demo.html  # Standalone client-side demo
```

## 🚀 Getting Started

### Prerequisites
- Python 3.9 or later
- pip

### Installation

```bash
git clone https://github.com/<your-username>/cyberguard.git
cd cyberguard
python3 -m venv venv
source venv/bin/activate       # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

A trained model (`url_classifier.pkl`) is already included, so you can run the
app immediately — no training step required.

### Run

```bash
python3 app.py
```

Then open **http://127.0.0.1:5050** in your browser.

### (Optional) Rebuild the dataset and retrain

```bash
python3 generate_dataset.py    # writes data/url_dataset.csv (8,000 labelled URLs)
python3 train_model.py         # trains the Random Forest, writes url_classifier.pkl + data/metrics.json
```

## 💻 Usage

### Via the web UI
Paste any URL into the input box and click **Scan URL** — or click one of the example chips to try a pre-loaded safe or malicious link.

### Via the API

**Safe / whitelisted example:**
```bash
curl -X POST http://127.0.0.1:5050/scan \
  -H "Content-Type: application/json" \
  -d '{"url": "https://www.github.com/anthropics"}'
```
```json
{
  "verdict": "Safe",
  "confidence": 99.9,
  "url": "https://www.github.com/anthropics",
  "domain": "www.github.com",
  "reason": "Domain matched trusted whitelist entry",
  "protocol": "HTTPS (secure)",
  "flagged": [],
  "response_ms": 0.3,
  "whitelisted": true
}
```

**Threat example:**
```bash
curl -X POST http://127.0.0.1:5050/scan \
  -H "Content-Type: application/json" \
  -d '{"url": "http://secure-login-verify-paypal.tk/account"}'
```
```json
{
  "verdict": "Threat Detected",
  "confidence": 97.0,
  "url": "http://secure-login-verify-paypal.tk/account",
  "domain": "secure-login-verify-paypal.tk",
  "protocol": "HTTP (insecure)",
  "flagged": ["Insecure protocol (HTTP, no TLS)"],
  "response_ms": 14.2,
  "whitelisted": false
}
```

**Invalid input:**
```bash
curl -X POST http://127.0.0.1:5050/scan -H "Content-Type: application/json" -d '{"url": ""}'
# 400 {"error": "Please enter a URL to scan."}
```

## ⚙️ Configuration

| What | Where | Notes |
|---|---|---|
| Trusted domains | `TRUSTED_DOMAINS` set in `app.py` | Add any domain you want to always mark Safe |
| Decision threshold | `HAZARD_THRESHOLD` in `app.py` | Default `0.5`; raise it to reduce false positives, lower it to catch more borderline cases |
| Server port | bottom of `app.py` | Default `5050` |
| Feature set | `feature_extraction.py` | Add new features here, then re-run `train_model.py` |

## 📊 Model Details

Trained on an 8,000-URL labelled dataset (balanced legitimate / phishing-style, 80/20 train-test split), using a `RandomForestClassifier` (`n_estimators=200`, `max_depth=12`).

| Metric | Score |
|---|---|
| Accuracy | **97.50%** |
| Precision | **99.10%** |
| Recall | **95.87%** |
| F1-Score | **97.46%** |
| Avg. response time (end-to-end) | **~12 ms** |

**Confusion matrix** (test set, n=1,600):

| | Predicted Safe | Predicted Threat |
|---|---|---|
| **Actual Safe** | 793 (TN) | 7 (FP) |
| **Actual Threat** | 33 (FN) | 767 (TP) |

**Feature importance:**

`is_https` 40.1% > `special_chars` 36.0% > `url_length` 10.4% > `domain_length` 10.0% > `has_ip_address` 3.6%

**On the dataset:** legitimate examples are built from ~80 real, well-known domains plus modern hyphenated/indie-style domains (`.io`, `.dev`, `.app`); phishing-style examples are synthesised using patterns well documented in phishing research (raw IP hosts, brand typosquatting, suspicious free TLDs, credential-harvesting keywords, `@`-symbol tricks, HTTP-only links). See `generate_dataset.py` for the full generation logic. For production use, swap this out for a real, continuously updated labelled dataset (e.g. PhishTank + a Common Crawl sample of legitimate URLs).

## 🧪 Testing

The following functional test cases are exercised against the live app (see the project report for the full table):

| Input type | Example | Expected |
|---|---|---|
| Whitelisted domain | `https://github.com/...` | Safe |
| Raw IP host | `http://192.168.44.10/login` | Threat Detected |
| Typosquat + suspicious TLD | `http://secure-paypa1-login.tk` | Threat Detected |
| `@` redirect trick | `http://acc-verify@login-bank.co` | Threat Detected |
| Empty input | `""` | 400 — rejected |
| Malformed URL | `htp:/badurl,com` | 400 — rejected |
| Non-whitelisted but benign | `https://my-startup-site.dev` | Safe |

Run them yourself with `curl` (see [Usage](#-usage)) once the server is running.

## ⚠️ Limitations

- Relies purely on the URL string — it does **not** inspect destination page content, so a legitimate domain compromised to host a phishing page will not be flagged.
- With only five lexical features, unusual-but-legitimate domains (e.g. long, multi-hyphenated startup names) can occasionally be flagged as threats.
- The whitelist must be maintained manually.
- Trained on a synthetic dataset for demonstration; real-world deployment should use a continuously updated dataset of confirmed phishing/legitimate URLs.
- This is an academic/portfolio project, not a hardened production security tool — do not rely on it as a sole line of defense.

## 🗺️ Roadmap

- [ ] Browser extension for pre-click link scanning
- [ ] DNS / WHOIS-based features (domain age, registrar reputation)
- [ ] Character-level deep learning model (LSTM/Transformer) for subtler lexical patterns
- [ ] IDN homograph detection (look-alike Unicode domains)
- [ ] User feedback loop to continuously improve the training set
- [ ] Public REST API + companion mobile app

## 🤝 Contributing

Contributions, issues, and feature requests are welcome.

1. Fork the repo
2. Create a branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'Add some feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

## 📄 License

Released under the [MIT License](LICENSE) — free to use, modify, and distribute.

## 🙏 Acknowledgments

- [scikit-learn](https://scikit-learn.org/) — machine learning library
- [Flask](https://flask.palletsprojects.com/) — web framework
- [PhishTank](https://phishtank.org/) / [Google Safe Browsing](https://safebrowsing.google.com/) — inspiration for the blacklist-vs-heuristic problem framing

---

<div align="center">

Built as a BCA final-year project — **CyberGuard // URL Threat Detector**

</div>
