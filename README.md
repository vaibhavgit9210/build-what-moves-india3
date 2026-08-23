# CivicFlow: a citizen-first Learner's Licence journey

A working prototype that redesigns the Karnataka Learner's Licence application, replacing "Which Sarathi menu do I need?" with "What do I need to do next?". Every screen answers three questions: where am I, what do I need to do, and what happens next.

**Live demo:** https://vaibhavkumar.is-a.dev/build-what-moves-india3/

**Prototype only.** Everything is fictional. It is not connected to Parivahan, Sarathi, Aadhaar, UIDAI, any payment gateway, any RTO, or any government service. Never enter real personal or payment details.

## Run it

Open `index.html` in any modern browser. No build step, no dependencies, no internet needed. Or serve the folder with any static server.

### Demo account

- Email: `aarav.demo@example.com`
- Password: `Demo@123` <!-- gitleaks:allow -->

Signing in resumes a mid-journey application (at Review), demonstrating return-user continuity. You can also start a fresh application without signing in; progress persists in `localStorage` across refreshes either way.

### Demo shortcuts (for reviewers and screenshots)

Query parameter before the hash: `index.html?e2e=<state>#/route`

| `e2e=` | Seeds |
|---|---|
| `reset` | wipes stored state |
| `login` | signed in, no application |
| `draft` | application complete through Documents (current step: Review) |
| `uncertain` | payment stuck in the uncertain state |
| `paid` | payment received, application number assigned |
| `passed` | full journey complete, licence ready |

Examples: `?e2e=uncertain#/apply/payment` · `?e2e=passed#/apply/licence` · `?e2e=paid#/dashboard`

## What it demonstrates

1. **Persistent progress model:** a 13-step stepper with four states (Completed, Current, Not started, Action required), a percentage, and a return-visit dashboard whose centrepiece is the *next action*, never a menu.
2. **Plain language first:** "What do you want to drive?" with everyday choices; MCWOG/MCWG/LMV codes are recorded behind the scenes and surfaced as secondary detail.
3. **Requirements up front:** the Before-you-start preflight lists time, documents, mobile and camera needs, and the indicative fee, with an explicit note that state rules vary (online testing is state-dependent; nothing is hard-coded as all-India).
4. **RTO by address, with recovery:** PIN-to-RTO lookup explains *why* the RTO is needed; an unmatched PIN offers check-the-PIN, manual selection, and report-missing-location instead of "contact your office".
5. **Identity without real identity:** a fake Aadhaar eKYC route (fictional Virtual ID, on-screen demo OTP, plain-language consent) plus an alternate document route. eKYC dynamically removes the photo upload from Documents.
6. **Editable review before payment**, with the vehicle-class choice called out as a high-impact confirmation.
7. **Three honest payment states:** ready, received, and uncertain. The uncertain state leads with **"Don't pay again yet."** and makes *Check payment status* the primary recovery action.
8. **Test readiness preflight:** six simulated device checks with exact fixes on failure; only the failed check re-runs and the application is never restarted.
9. **Practice directly before the real test;** the real test keeps failure recovery in the journey: retest fee and next slot together, one button, no separate services.
10. **Licence download as the dominant final action,** with a document preview, file download, and print.

## Accessibility (WCAG 2.2 AA patterns)

Skip-to-content link, semantic landmarks and headings, full keyboard operation, visible focus rings, labels tied to every field, error summary linked to fields (focus moves to it on failed submit), 44px minimum targets, status conveyed by words and symbols rather than colour alone, text-size controls plus reflow-safe layout at 200% zoom, `prefers-reduced-motion` respected, a polite live region announcing route and status changes, plain-language explanations of bureaucratic terms, and no CAPTCHA. No personal data ever appears in URLs.

## Architecture

Single self-contained `index.html` (the deliberate no-build variant the brief allows), organised internally like the suggested module layout:

- **CONTENT**: versioned demo Karnataka configuration (fees, RTO list, PIN map, help articles, question bank). All state-rule claims come from here.
- **RULES**: deterministic services: age/eligibility, vehicle-to-category mapping, PIN-to-RTO mapping, fee breakdown.
- **STATE**: one application state machine (`steps`, `data`) persisted to `localStorage`; no separate unrelated pages.
- **ROUTER**: hash routes carrying only route names, never data.
- **VIEWS/COMPONENTS**: reusable render functions: header controls, stepper, choice cards, fields, inline help, status panels, sticky action bar, payment states, readiness checks, question cards, licence preview.
- **Optional AI, by design absent from the core:** the "Explain this" assistant on each step answers only from the approved versioned content above. An LLM (if added) would sit behind a retrieval layer server-side and would never decide eligibility, fees, categories, RTO, payment status or identity.

## Production hardening (required before any real deployment)

- **Security:** move all state config and rules server-side as the source of truth; real authentication (with rate limiting and account recovery); CSRF protection; strict CSP; server-side validation of every field; signed licence documents; API keys (payment, eKYC, any LLM) only on the server; prompt-injection defences, rate limits and canned fallbacks for any assistant.
- **Privacy:** real Aadhaar flows must go through UIDAI-approved AUA/KUA channels with consent artefacts and data-minimisation; encrypt PII at rest; retention schedules; no PII in URLs, logs or analytics.
- **Accessibility:** a full WCAG 2.2 AA audit with assistive-technology users; Hindi and Kannada localisation; content tested for plain language.
- **State-rule configuration:** per-state config service for fees, documents, online-test availability, age rules and RTO mappings, with versioning and effective dates; the payment provider's separate fee/retest/verify/receipt services unified behind one status API.
- **Audit and operations:** append-only audit log of application transitions; reconciliation jobs for uncertain payments; monitoring for stuck applications; a real missing-location reporting pipeline.
- **API integration:** Sarathi/Parivahan application APIs, payment gateway with idempotency keys (the true fix for double payment), test-proctoring service, and DigiLocker for issued licences.
