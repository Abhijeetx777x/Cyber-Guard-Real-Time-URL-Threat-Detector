<div align="center">

# 🛡️ CyberGuard // URL Threat Detector

**Real-time phishing URL detection using structural analysis — not stale blacklists.**

[![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-3.0-000000?logo=flask&logoColor=white)](https://flask.palletsprojects.com/)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-1.5-F7931E?logo=scikitlearn&logoColor=white)](https://scikit-learn.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

[Overview](#-overview) •
[Demo](#-demo) •
[How it works](#-how-it-works) •
[Getting started](#-getting-started) •
[Performance](#-model-performance) •
[Project structure](#-project-structure) •
[Roadmap](#-roadmap)

</div>

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

## ✨ Features

- **Zero-day capable** — flags never-before-seen phishing domains using structure alone, no blacklist required
- **Sub-50ms verdicts** — whitelist check + feature extraction + inference typically completes in ~12 ms end-to-end
- **Explainable output** — every "Threat Detected" verdict lists the specific structural red flags that triggered it
- **Trusted-domain whitelist** — well-known domains short-circuit straight to "Safe" to avoid false alarms
- **Clean REST API** — a single `POST /scan` endpoint returns JSON, easy to wire into other tools
- **Dark, glassmorphic UI** — single-page front end with animated result cards, no framework required

## ⚙️ How it works

```
User Interface  →  Whitelist Filter  →  Feature Extraction  →  Random Forest  →  Result Card
  (paste URL)      (trusted domains)    (feature_extraction.py)   (url_classifier.pkl)  (<50ms)
```

1. **Whitelist filter** — if the domain matches a curated list of trusted, high-traffic domains, the URL is marked Safe immediately.
2. **Feature extraction** (`feature_extraction.py`) — otherwise, five structural features are computed straight from the URL string:

   | Feature | Description |
   |---|---|
   | `url_length` | Total character length of the URL |
   | `domain_length` | Character length of the domain/host |
   | `special_chars` | Count of `.` `-` `@` characters |
   | `has_ip_address` | Whether the host is a raw IP address instead of a domain |
   | `is_https` | Whether the URL uses HTTPS |

3. **Classification** — the feature vector is passed to a pre-trained **Random Forest Classifier** (`url_classifier.pkl`), which returns a hazard confidence score.
4. **Result** — a colour-coded verdict (🟢 Safe / 🔴 Threat Detected) is rendered instantly, along with the reasoning behind it.

## 🚀 Getting started

### Prerequisites
- Python 3.9+

### Installation

```bash
git clone https://github.com/<your-username>/cyberguard.git
cd cyberguard
python3 -m venv venv
source venv/bin/activate       # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

A trained model (`url_classifier.pkl`) is already included, so you can run the
app immediately. To regenerate the dataset and retrain from scratch:

```bash
python3 generate_dataset.py    # writes data/url_dataset.csv (8,000 labelled URLs)
python3 train_model.py         # trains the Random Forest, writes url_classifier.pkl
```

### Run

```bash
python3 app.py
```

Open **http://127.0.0.1:5050** in your browser.

### API

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

## 📊 Model performance

Trained on an 8,000-URL labelled dataset (balanced legitimate / phishing-style,
80/20 train-test split):

| Metric | Score |
|---|---|
| Accuracy | **97.50%** |
| Precision | **99.10%** |
| Recall | **95.87%** |
| F1-Score | **97.46%** |
| Avg. response time (end-to-end) | **~12 ms** |

**Feature importance** (what the model actually relies on):

`is_https` (40.1%) > `special_chars` (36.0%) > `url_length` (10.4%) > `domain_length` (10.0%) > `has_ip_address` (3.6%)

> ⚠️ **Known limitation:** with only five lexical features, unusual-but-legitimate
> domains (e.g. long, multi-hyphenated startup names) can occasionally be
> flagged. This is an inherent trade-off of a purely structural approach — see
> [Roadmap](#-roadmap) for planned mitigations.

## 📁 Project structure

```
cyberguard/
├── app.py                  # Flask server — routes: / and /scan
├── feature_extraction.py   # URL → structural feature vector
├── train_model.py          # Trains the Random Forest, saves url_classifier.pkl
├── generate_dataset.py     # Builds data/url_dataset.csv
├── url_classifier.pkl      # Pre-trained model
├── requirements.txt
├── data/
│   ├── url_dataset.csv     # 8,000 labelled training URLs
│   └── metrics.json        # Latest training run metrics
├── templates/
│   └── index.html
├── static/
│   ├── css/style.css       # Dark glassmorphic theme
│   └── js/script.js
└── screenshots/
```

## 🗺️ Roadmap

- [ ] Browser extension for pre-click link scanning
- [ ] DNS / WHOIS-based features (domain age, registrar reputation)
- [ ] Character-level deep learning model (LSTM/Transformer) for subtler lexical patterns
- [ ] IDN homograph detection (look-alike Unicode domains)
- [ ] User feedback loop to continuously improve the training set
- [ ] Public REST API + companion mobile app

## 🧰 Tech stack

`Python` · `Flask` · `scikit-learn` · `pandas` · `NumPy` · `HTML5` / `CSS3` / `JavaScript`

## 📄 License

Released under the [MIT License](LICENSE).

## 👤 Author

Built as a BCA final-year project — **CyberGuard // URL Threat Detector**.

