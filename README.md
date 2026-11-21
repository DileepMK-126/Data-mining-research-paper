ScamShield AI
LLM-based scam and phishing analysis with explainable, evidence-linked risk reports.
ScamShield AI is a web tool that helps people judge whether a fake job offer, forged offer letter, rental deposit request, phishing email, SMS or Telegram/WhatsApp chat looks fraudulent, and explains the risk in plain words.
> **Status: research prototype (v2.6).** This project is a design proposal with a working front-end prototype. It has **not** been evaluated on a labelled dataset, and no accuracy figures are claimed. Do not rely on it as a proven detector.
---
Core idea
The tool follows one rule: it may only say what it can back up with a saved observation.
Every finding is stored in an evidence log with its source, time and status. The language model may only reword those records. It cannot add new facts. Every factual sentence in a report points to a saved record.
Each report keeps four things separate:
Part	Question it answers
Content risk	How risky does the message look?
Email authentication	Was the sender's email technically verified (SPF / DKIM / DMARC)?
Identity verification	Was the company or recruiter independently confirmed?
Uncertainty	What could not be checked, and what would help?
A low risk band never means "verified legitimate", and a passing authentication result never means "harmless".
Features (current prototype)
Two modes: Job Offer / Recruiter, and Rental / Deposit Fraud
Four input methods: pasted text, website link, PDF upload, screenshot (OCR)
Accepted content: job offers, recruiter messages, emails, WhatsApp texts, Telegram chats, appointment letters
Scam Threat Index (0–100) with a risk label
Weighted breakdown across seven areas:
payment risk, URL/domain risk, job or rental anomaly, identity mismatch, social engineering, credential and data risk, document forensics
Pattern-match panel for known scam types
Highlighted red flags
Domain and phishing panel
Company and recruiter identity panel
Ranked protective steps, including the national cyber crime helpline (1930)
Seven-item verification checklist
Downloadable PDF report
One-click demo scenarios, scan history, dashboard, threat feed, voice assistant and advisor chat
Known limitations
The prototype does not yet parse raw email headers or check SPF, DKIM or DMARC. This is planned for the next phase.
The scoring weights have not been learned or validated on held-out data, so the 0–100 number should be read as a rough indicator only.
A low score can be misread as "safe". In the paper's missed-call SMS test, the prototype returned 4/100 with a low-risk label even though the claimed company was unverified. The design's separate fields are meant to fix this.
No claims are made about the backend models or fraud-sample database mentioned in the interface.
Architecture (proposed design)
```
1. Input intake            SMS / chat / email / link / PDF / screenshot
2. Normalise + provenance  original fields kept apart from extracted ones
3. Modality gate           which checks are valid for this input?
4. Run eligible checks     language | link | document | email auth
5. Evidence log            source, time, status, provenance
6. Explanation + claim     every factual claim linked to a record
   validation
7. Report                  risk | authentication | identity | uncertainty
```
Modality gate
Each check ends in one of four states: applicable, not applicable, unavailable or failed. A missing requirement is never turned into a failure.
Input	Checks that make sense	Must not be claimed
SMS or chat	Wording, money/credential requests, link and domain analysis	Any SPF, DKIM or DMARC result
Raw email with headers	Wording, links, SPF/DKIM/DMARC with alignment	That a pass means content is harmless
PDF or offer letter	Text extraction, consistency, contact-domain checks, company lookup	That a logo or letterhead proves it is genuine
Screenshot	OCR text, visible link text, consistency checks	Real redirect path, hidden metadata, missing headers
Standalone link	Redirect check in a controlled service, look-alike and reputation checks	Intent or sender of a message not provided
Evidence modules
Language: fee/deposit/password/OTP/bank/ID requests, guaranteed selection, urgency, mismatched company names, moves to unverified channels
Link: real destination, look-alike domains, shorteners, reputation data
Document: cross-checks names, dates and contacts; marks OCR-derived values
Email authentication: uses results only from a trusted receiving server or controlled verifier; otherwise reports "not verifiable from supplied input"
Evidence record fields
`evidence_id`, `case_id`, `feature_family`, `raw_observation`, `normalised_value`, `source_type`, `source_reference`, `observed_at`, `retrieved_at`, `verification_status`, `confidence`, `applicable_modality`, `interpretation_type`
Scoring (specification)
```
C = min(100, max(0, Σ wᵢ · gᵢ(xᵢ)))
```
`xᵢ` is a normalised risk feature, `wᵢ` its preset weight and `gᵢ` a bounded transformation. Authentication and identity results stay outside `C`. When input is poor (bad OCR, failed lookups, out-of-distribution sample), the tool should abstain.
Status vocabulary
Status	Meaning
`PASS` / `FAIL`	Verifier ran and returned a result for the stated identity
`TEMPERROR` / `PERMERROR`	Check could not finish, briefly or due to a lasting fault
`NONE`	Relevant policy or signature record does not exist
`NOT_AVAILABLE`	Check could apply, but needed evidence was not supplied
`NOT_APPLICABLE`	Check makes no sense for this input type
Design principles
Provenance – every value is marked as original or extracted
Separation – cryptographic results and language judgements stored and shown apart
Plain explanations – everyday words, technical detail one click away
Honest gaps – missing data is shown as missing, not as pass or fail
Injection resistance – text inside a message is never followed as an instruction
Repeatability – versions, thresholds and prompts are fixed and logged
Security and privacy
System instructions are kept separate from user content; model output follows a fixed format
Link checking runs in a controlled service that blocks private address ranges and unsafe redirects (SSRF mitigation); links and attachments are never opened automatically
All outside lookups are time-stamped; failed lookups are logged, never filled in by the model
Sensitive values are masked in exports; stored files should be encrypted with set deletion periods
Users should be told whether a third-party model provider receives their content
Every blocklist entry carries its source, date and review status
Evaluation plan
No experiments have been run yet. The planned evaluation compares five setups on the same samples:
Rules and simple text features
Classical text classifier (e.g. TF-IDF + logistic regression / SVM)
LLM-only classifier
LLM with link and domain checks
Full design (gate + evidence log + claim validation)
Ablations remove one component at a time. Metrics cover precision/recall/F1, PR-AUC, Brier score and calibration, abstention and OCR failure rates, explanation grounding (share of claims backed by a log entry), and user decision accuracy. A consented user study will compare a label-only interface with the evidence-linked report.
Roadmap
Build and test header parsing and SPF/DKIM/DMARC verification on emails with known ground truth
Collect a labelled multi-format dataset, including a later time slice for drift testing
Learn or validate risk weights and calibrate scores
Run the user study and publish all results, including failures
Ethical notes
False alarms can damage genuine recruitment or payroll exchanges, and missed scams can cost people their savings. Reports use measured language, avoid calling any person or company fraudulent on weak evidence, and allow human review when the tool is unsure. "No information found" is never shown as a negative finding, since new and small businesses often have little online presence.
If you think you are being scammed (India)
Report to the National Cyber Crime Helpline at 1930 or at the national cyber crime reporting portal.
Authors
Manoj H. G. – manojhg321@gmail.com
Dileep M. K. – dileep.m.k126@gmail.com
Chinmay C. – chinmaychoudhari620@gmail.com
Reference
Large Language Model-Based Phishing Email Detection and Explainable Risk Assessment. See the accompanying paper for full references (RFC 7208, RFC 6376, RFC 9989, NIST SP 800-177 Rev. 1, and related work on LLM phishing detection and explainability).
License
Not yet specified. Add a license before public release.
