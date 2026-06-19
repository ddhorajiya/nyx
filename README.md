# nyx

**Autonomous coding agent that ships reviewed diffs while you sleep — and never pushes on its own.**

Nyx picks up queued backend tickets overnight, writes the change, runs the full test suite, and leaves a review-ready diff in chat. Autonomy stops at the diff: a human approves every merge.

> Personal project · built & operated by one engineer.
> This repository is the public write-up + showcase site. The implementation is kept private.

**Live:** _add your GitHub Pages / Netlify URL here_

---

## what it is

A pipeline + chat assistant that turns an issue queue into test-verified, review-ready code changes — built with the reliability, security, and verification controls needed to trust an autonomous agent running unattended.

The interesting part isn't that an LLM writes code. It's the scaffolding that makes that safe and reliable enough to run while you sleep.

## how it works

```
  cron / chat
      |
      v
 +-----------+   +-----------+   +-----------+   +-----------+   +-----------+
 | scheduled |-->|   agent   |-->|   build   |-->| verify &  |-->|  review   |
 |  trigger  |   | implements|   |  & test   |   |   guard   |   |  in chat  |
 +-----------+   +-----------+   +-----------+   +-----------+   +-----------+
   pull queue     trace + edit    full suite     tests pass?      diff + log
                                                 no publish       human merges
```

- **scheduled trigger** — wakes on a schedule (or on demand from chat) and pulls the assigned ticket queue.
- **agent implements** — a headless LLM coding agent traces the code and makes the change.
- **build & test** — compiles and runs the full test suite in isolation.
- **verify & guard** — confirms the tests actually pass; enforces the no-publish guardrails.
- **review in chat** — posts a reviewable diff + run log; a human approves the merge.

## trust model

An agent that writes code is the easy 20%. The 80% is making it safe and reliable enough to run unattended. These are the guardrails:

| control | what it does |
|---|---|
| **verify, don't trust** | parses the real build/test output — a model *claiming* success is not success |
| **least privilege** | sandboxed: no network egress, cannot publish code, cannot read secrets |
| **propose, never publish** | every change left uncommitted for review; version-control state is hard-verified, with alerts on drift |
| **dead-man's switch** | a scheduled health check flags silent failures — a missing result is itself an alarm |
| **two-way chatops** | reports each run and takes new instructions from chat, multi-turn, from anywhere |
| **self-review loop** | re-reads its own diff for missing tests, edge cases, and docs before handing off |

## a run, end to end

```
02:00  cron fired · budget cap set
02:01  TICKET-204 -> backend · paginate GET /orders
02:03  implementing · 6 files traced
02:09  + build ok
02:12  + 214 passed / 0 failed
02:13  ! self-review: my query dropped a soft-delete filter — reverted
02:15  + re-ran — 214 passed
02:15  guardrails  + not pushed   + not committed   + within budget
02:16  -> diff posted to chat
08:41  human: why no DB migration?
08:41  nyx:   none needed — params are query-only, no schema change
08:43  human: merging
```

- **it caught itself** — the self-review reverted its own over-broad query before review.
- **it stayed in its lane** — no push, no commit, no unjustified schema change, under a hard budget cap.
- **you stayed in control** — it answered a question, then waited. The merge was a human's.

## stack

Headless LLM coding agent · PowerShell · scheduled automation · issue-tracker & chat APIs · JVM service / Maven test suite · Git.

## design notes

A few decisions worth calling out (happy to discuss any of them):

- **Verification over trust.** The system never accepts the agent's word that a change works — it runs the real build and tests and parses the results. Success is an observed fact, not a claim.
- **Bounded blast radius.** The agent operates in a least-privilege sandbox and is structurally unable to publish: the worst a bad run can do is waste a run.
- **Fail loud, including on silence.** A separate scheduled check treats the *absence* of a result as a failure, so a crashed run can't pass quietly.
- **Human-in-the-loop by design.** Autonomy stops at the diff. Everything is left for review; nothing ships without a human.

## status

Personal project, in daily use. The implementation is kept private; this repo is the architecture write-up and the showcase site (`index.html`).

**Contact:** deepdhorajiya2@gmail.com

## license

MIT — see [LICENSE](LICENSE). Applies to this write-up and the showcase site; no implementation is included.
