# Multi-line Insurance Policy and Claims Management System

A single platform that manages **motor, health, life, property and travel insurance** together. One customer profile can hold many policies, and every claim follows one common workflow: registration, policy and cover validation, document verification, fraud screening, decision and settlement.

This repository contains the project report (PDF) and the Python prototype of the claims decision engine described in it.

---

## Team

| Role | Name |
|------|------|
| Team Leader | Deeba Prabha K |
| Team Member | Pandiselvi D |
| Team Member | Thanaselvi T |
| Team Member | Jegatheeshwari A |
| Team Member | Nirmalkumar M |

---

## Problem

When each line of business runs on its own software:

- The same customer is stored many times with different details.
- KYC documents have to be submitted again for every new policy.
- Claim status is hard to track across systems.
- Management cannot see one customer's total exposure or premium.
- Manual checks are slow and open to human error and fraud.

## Solution

One platform with one customer record linked to many policies. All claims pass through the same automated checks, every decision stores a reason, and business rules (such as the auto-approval limit) are kept in one place so they can be changed easily.

| Feature | Existing system | Proposed system |
|---------|-----------------|-----------------|
| Customer data | Duplicated in each line | One customer, many policies |
| Claim checking | Manual and slow | Automated rule checks |
| Small claims | Wait for officer | Auto-approved within limit |
| Fraud control | Individual judgement | Rule-based scoring |
| Reports | Separate per line | Combined for all lines |
| Audit trail | Paper or scattered | Status and reason stored |

---

## Features

- Customer registration with KYC stored once
- Policy creation in any line of business (status: `ACTIVE`, `LAPSED`, `CLOSED`)
- Premium and payment recording
- Claim registration with amount and reason
- Validation that the policy is active and the claim is within remaining cover
- Automatic approval of small claims, surveyor routing for large ones, rejection with a reason
- Rule-based fraud scoring
- SQL reports on claims, premium and claim ratio

## Lines of business

| Line | Cover provided |
|------|----------------|
| Motor | Own damage, third-party liability, theft of vehicle |
| Health | Hospital expenses, cashless treatment, critical illness |
| Life | Death benefit, maturity benefit, term cover |
| Property | Fire, flood, burglary, natural calamities |
| Travel | Trip cancellation, baggage loss, medical emergency abroad |

## Stakeholders

| Stakeholder | Role |
|-------------|------|
| Customer | Buys policies, pays premium, raises claims, tracks status |
| Agent | Collects proposals, helps choose products, assists renewals and claims |
| Claims officer | Verifies documents, reviews fraud score, approves or rejects claims |
| Surveyor | Inspects large losses and recommends the settlement amount |
| Administrator | Manages products, users, roles, business rules and reports |

Indirect stakeholders include the regulator (IRDAI), management and finance, network hospitals and garages, nominees and fraud investigators.

---

## How it works

### Policy lifecycle

`Enquiry` → `KYC and Proposal` → `Underwriting` → `Premium Payment` → `Policy Issued` → `Renewal / Lapse`

### Claims lifecycle

`Claim Registered` → `Policy and Cover Check` → `Documents Verified` → `Fraud Screening` → `Decision`

The decision is one of:

- **Approved** automatically (amount up to the limit)
- **Surveyor review** (amount above the limit)
- **Rejected** with a reason (policy inactive, or amount exceeds cover)

### Architecture

Layered design: **Users** → **Presentation** (web portal, mobile app, dashboard) → **Business** (policy, premium, claims, fraud check, notification) → **Data** (customers, policies, claims, payments, documents).

### Database

Four main tables: `customer`, `policy`, `claim` and `payment`. One customer has many policies, one policy has many claims, and one claim has one or more payments. The SQL schema and a report query are in Section 6.4 of the report.

---

## Python prototype

The prototype uses three classes:

| Class | Purpose |
|-------|---------|
| `Policy` | Stores policy details and the amount already claimed |
| `Claim` | Stores one claim request |
| `ClaimsManager` | Applies the decision rules in a fixed order |

### Decision rules (in order)

1. If the policy is not `ACTIVE`, reject (policy inactive).
2. If the claim amount is greater than the remaining cover (`sum_insured - claimed`), reject (exceeds cover).
3. If the amount is at most **20,000**, approve automatically and add it to `claimed`.
4. Otherwise, send for surveyor review.

The order matters: a claim on a lapsed policy is rejected before any other rule is considered.

### Run it

Requires Python 3.8 or higher. Save the code from Section 6.1 of the report as `claims_manager.py`, then run:

```bash
python claims_manager.py
```

### Sample output

```
C1 Motor 15000 -> APPROVED (auto)
C2 Health 80000 -> SENT FOR SURVEYOR REVIEW
C3 Health 900000 -> REJECTED (exceeds cover)
```

---

## Fraud detection

A simple rule-based score. If the total is **50 or more**, the claim goes to a human investigator instead of being auto-approved.

| Rule | Reason | Points |
|------|--------|--------|
| Early claim | Claim made within 30 days of policy start | 30 |
| Frequent claims | More than 3 claims in the last 12 months | 30 |
| High amount | Claim above 80% of sum insured | 25 |
| Duplicate document | Same bill number used in another claim | 40 |

## Testing

| No. | Test case | Expected result |
|-----|-----------|-----------------|
| 1 | Small valid claim (Motor, 15,000) | `APPROVED (auto)` |
| 2 | Large valid claim (Health, 80,000) | `SENT FOR SURVEYOR REVIEW` |
| 3 | Claim above cover (Health, 900,000) | `REJECTED (exceeds cover)` |
| 4 | Inactive policy | `REJECTED (policy inactive)` |
| 5 | Cover exhausted by earlier claims | `REJECTED (exceeds cover)` |

Cases 1 to 3 were run and verified in the program output. Cases 4 and 5 follow from the rule order and should be added as unit tests before real use.

## Analytics

The report includes charts (claims by line, claim outcome split, monthly premium vs claims paid). These use **sample data prepared for illustration**, not real company figures.

---

## Technology

| Item | Requirement |
|------|-------------|
| Language | Python 3.8+ |
| Database | MySQL / PostgreSQL / SQLite |
| Tools | VS Code or PyCharm, Git, a web browser |
| OS | Windows, Linux or macOS |

## Repository structure

```
.
├── README.md
├── claims_manager.py                              # Python prototype (code from report Section 6.1)
└── Multi-line_Insurance_Project_Report_v4.pdf     # Full project report
```

## Limitations

- Premium calculation is simplified.
- No real payment gateway or insurer API integration.
- Fraud detection is rule-based only, with no machine learning.
- Real-world insurance regulations are not fully modelled.
- This is an academic prototype, not production software.

## Future scope

- AI-based assessment of photos, bills and reports
- Chatbot support for registration and claim tracking
- Blockchain-based tamper-proof records
- Telematics and IoT-based premium pricing
- Mobile-first claims with photo upload
- Predictive analytics for claim volume and pricing
- Payment gateway, OCR, notifications, role-based security and a full user interface

## References

- Insurance Regulatory and Development Authority of India (IRDAI), official website and regulations
- The Insurance Act, 1938 (India)
- Python Software Foundation, Python 3 documentation
- Elmasri, R. and Navathe, S., *Fundamentals of Database Systems*, Pearson
- Official documentation of MySQL / PostgreSQL
