# build-what-moves-india3 (CivicFlow)

Prototype redesign of the Karnataka Learner's Licence journey (Sarathi), built from
the brief in `../build-what-moves-india2/civicflow-prototype/CLAUDE_PROMPT.md`.
Repo `vaibhavgit9210/build-what-moves-india3`, live at
https://vaibhavkumar.is-a.dev/build-what-moves-india3/ (Pages from `gh-pages`,
so deploy = push BOTH `main` and `main:gh-pages`).

## Shape

Single-file `index.html`, no build step (house style; the brief's fallback option,
chosen deliberately to avoid the CI-deploy machinery bwmi2 needed). Script is
organised into CONTENT (versioned demo KA config: fees, RTO/PIN map, help
articles, question bank) / RULES (deterministic age, vehicle-category, RTO, fee
functions) / STATE (one application object + `steps` map in localStorage key
`civicflow.v1`) / hash ROUTER (`#/`, `#/signin`, `#/dashboard`, `#/apply/<step>`,
never any data in URLs) / VIEWS / delegated ACTIONS.

- 13 steps, statuses done/action/current/not-started; `currentStepId()` = first
  non-done step; later steps are locked in the router.
- Payment has 4 internal statuses (`ready/processing/uncertain/received` plus a
  transient `checking`); "uncertain" sets `steps.payment='action'` and drives the
  "Don't pay again yet" recovery. A demo radio picks the outcome.
- Test fail sets `steps.test='action'`; retest = pay fee + slot in one action,
  then `start-retest` resets the test sub-state and bumps `attempt`.
- Demo login `aarav.demo@example.com` / `Demo@123` (deliberately public;
  source lines carry `gitleaks:allow` for the machine's gitleaks shim).
  Signing in with no stored application seeds a draft (resume-at-Review demo).
- Demo OTP is `123456`, shown on screen. Never collects real identity data.

## Test hooks (query BEFORE the hash)

`?e2e=reset|login|draft|uncertain|paid|passed` then any `#/route`.
E.g. `?e2e=uncertain#/apply/payment`, `?e2e=passed#/apply/licence`.
Pages are rAF-free; headless Chrome with `--virtual-time-budget=4000
--timeout=12000` works, but Chrome hangs on exit: poll for the PNG then
`pkill -f headless=new`. Analytics beacon (va_sid block) is before `</body>`
and self-disables on file://.

## Rules that bit before (respect them)

- No em/en dashes anywhere in copy (user rule; checked with a grep for `—`/`–`).
- The A− text-size button uses U+2212 minus, which is allowed.
- Push remotes only via the `github-personal` SSH alias.
