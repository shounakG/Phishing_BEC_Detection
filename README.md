# Phishing and BEC Email Detection
 
An open, reproducible study of detecting phishing and business email compromise (BEC) emails using public data, comparing classical machine learning with LLM-based classifiers, and testing how well both hold up against an adversary.
 
> **Disclaimer:** Independent personal project. It uses only publicly available data and is not affiliated with, or derived from, any employer's systems, data or code.
 
---
 
## Problem statement
 
Two kinds of malicious email are the focus of this project:
 
- **Phishing:** emails that try to trick a recipient into clicking a malicious link, opening an attachment or entering credentials.
- **Business email compromise (BEC):** fraud in which an attacker impersonates an executive, vendor or coworker and asks for a payment, gift cards, a change of bank details or sensitive data. A BEC message may contain no malicious link or attachment, so the text, the sender context and the request itself may be the only signals.
**Goal:** build and evaluate a classifier that labels an email as *safe*, *phishing* or *BEC*, and measure how reliable it is under three constraints:
 
1. **Low false-positive rate.** A false positive blocks a legitimate email, so models are judged at fixed false-positive rates (for example 1% and 0.1%) rather than by raw accuracy.
2. **Robustness to an adaptive attacker.** Performance is measured on rewritten and adversarial emails, not only on clean test data.
3. **Cost and speed.** A classical model and an LLM-based classifier are compared on accuracy, latency and cost per email, so the trade-offs are explicit.
### Research questions
 
1. How far can a simple, fast baseline (text features plus header and link features) get, and where does it fail?
2. Does an LLM classifier (prompted and/or fine-tuned) catch BEC and rewritten phishing that the baseline misses, and at what cost?
3. How much does performance drop under adversarial rewriting and prompt injection (emails that try to manipulate an LLM-based classifier)?
4. How much of the measured performance is real, and how much comes from dataset artifacts such as source, era or formatting?
### Scope
 
**In scope**
- English-language emails: headers (From, Reply-To, Subject, dates), body text and URLs.
- A baseline model, an LLM classifier and a shared evaluation harness.
- Adversarial tests: paraphrased attacks, obfuscation and prompt-injection attempts.
**Out of scope for the initial work**
- Attachments, images and QR codes.
- Any real organization's email, telemetry or labels.
- Building tools to *create* attacks. Adversarial examples are used only to measure and improve defenses.
**Possible later extensions**
- Deployment, real-time serving and user-facing tooling.
---
 
## Data
 
Only public datasets with terms that permit this use. Each dataset's license, source and collection details are verified and recorded in [`docs/datasets.md`](docs/datasets.md) before use.
 
| Role | Candidate source | To verify |
|---|---|---|
| Safe email | Enron email corpus | License, size, date range |
| Safe email | SpamAssassin public corpus (ham) | License, size, date range |
| Phishing | Nazario phishing corpus | License, size, date range, label quality |
| Phishing / spam | SpamAssassin public corpus (spam) | Whether spam labels match phishing |
| BEC | Synthetic (generated) | No public BEC dataset selected yet. Examples are generated, reviewed and clearly labeled as synthetic. |
 
**Do not commit raw data.** The `data/` directory is git-ignored. The repo contains download and preparation scripts only.
 
### Labeling
 
- Three classes: `safe`, `phishing`, `bec`.
- BEC examples are synthetic, so results on them are reported **separately** from results on real data and never mixed into a single headline number.
- A sample of each class is reviewed by hand to estimate label noise.
---
 
## Evaluation plan
 
**Primary metric:** recall at a fixed false-positive rate (1% and 0.1%).
 
**Secondary metrics:** precision-recall AUC, per-class precision and recall, confusion matrix, latency per email, and cost per 1,000 emails.
 
### Avoiding misleading results
 
If safe and malicious emails come from different sources, a model can learn the source (era, formatting, headers) instead of the attack. Safeguards:
 
- Normalize or strip headers, signatures and other source-specific artifacts, and report results with and without them.
- Split train and test **by source or by time**, not only at random.
- Include safe and malicious examples from the same source where possible.
- Report an "artifact-only" baseline (a model that sees only source-specific features) as a sanity check. If it scores well, the dataset has a leak.
### Adversarial tests
 
- Paraphrasing and tone changes to existing attacks.
- Obfuscation (character substitutions, spacing, unusual formatting).
- Prompt-injection text inside an email aimed at the LLM classifier.
Each is reported as the drop in recall at the fixed false-positive rate.
 
---
 
## Approach
 
1. **Baseline:** TF-IDF with logistic regression or gradient boosting, plus simple features such as sender and reply-to mismatch, urgency words, link count and domain features.
2. **LLM classifier:** a prompted model first, then optional fine-tuning. Compared with the baseline on the same splits and metrics.
3. **Evaluation harness:** one script that runs any model against the same data, splits and adversarial sets, and writes a results table.
---
 
## Repository layout
 
```
.
├── README.md
├── LICENSE
├── docs/
│   └── datasets.md        # source, license, size, date range for each dataset
├── data/                  # git-ignored; raw and processed data live here
├── notebooks/             # exploration
├── src/
│   ├── data/              # download and prep scripts
│   ├── models/            # baseline and LLM classifiers
│   └── eval/              # metrics and evaluation harness
├── results/               # tables and figures produced by the harness
└── .gitignore
```
 
## Setup
 
```bash
git clone <your-repo-url>
cd <repo-name>
python -m venv .venv && source .venv/bin/activate
pip install pandas scikit-learn jupyter
```
 
Dataset download and preparation instructions are added alongside each dataset's entry in `docs/datasets.md`.
 
## Responsible use
 
This project studies defenses. Synthetic and adversarial examples exist to measure and harden detection, not to help craft attacks. Do not use the contents of this repo to send deceptive email.
 
## Limitations
 
- Public email corpora may not reflect current attacks.
- BEC data is synthetic, so BEC results are indicative, not production-grade.
- Label noise in public datasets limits how well measured performance reflects real-world performance.
## License
 
Code is released under the MIT License (see `LICENSE`). Datasets keep their original licenses and are not redistributed in this repository.
