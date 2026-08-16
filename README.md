# AI Systems for IT Operations & Security

I build AI systems that give small teams back their attention.

Three production systems, designed and shipped solo, for an IT organization supporting a multi-brand franchise network. Each one takes work that was quietly consuming a team with no spare capacity — manual email routing, ticket reconstruction, more security findings than anyone had time to read — and turns it into something that runs unattended and can be trusted while it does.

Getting a model to produce output was the easy part. The work was making that output **safe enough to act on, cheap enough to run continuously, and honest enough that people kept using it after the novelty wore off.**

> **A note on code.** These are production systems built for my employer, so the source isn't public. What's here is the architecture, the design reasoning, and the trade-offs — the parts I'd want to talk through anyway.

---

## At a Glance

| | The problem | What it does | Stack | Outcome |
|---|---|---|---|---|
| **[Email Triage & Routing Agent](docs/01-email-triage-routing.md)** | 92% of a shared support mailbox belonged to an outside vendor, and reached them one manual forward at a time | Classifies each incoming request by ownership domain and routes it to the responsible team, unattended | n8n · Claude · Gmail API | Time-to-support cut from up to an hour down to under a minute; ~275 manual handoffs/month eliminated for ~$5/month |
| **[Service Desk Intelligence Agent](docs/02-service-desk-intelligence-agent.md)** | Every ticket cost minutes of re-explanation before real work started, and leadership had counts instead of a picture | Reads every open ticket nightly, leaves a grounded summary, next step, and blocker; feeds a three-cadence leadership dashboard | Python · Claude (vision) · Firestore · Looker · GCP | Higher technician follow-through; at-risk tickets surface automatically; KB gaps became a tracked metric with quarterly targets |
| **Attack Surface & Exposure Management Platform** | Scanner output arriving faster than anyone could read it, plus constant estate drift across cloud and physical locations, watched by a small team alongside everything else they own | Correlates three security vendors into the few findings worth a person's week, with evidence attached and provable scope discipline | Python · Postgres · Claude · GCP SCC · Shodan · KEV/EPSS | Level-one triage automated; inventory drift caught in the first weeks; every scan decision auditable six months later |

---

## The Projects

### 1. Email Triage & Routing Agent

A shared support mailbox received requests spanning two unrelated support domains — third-party AV equipment and internal IT systems — and every message landed in the same place. The internal desk opened each one, worked out who owned it, and manually forwarded the majority elsewhere. **About 92% of that volume belonged to the vendor.**

The agent classifies each message on arrival and forwards it to the responsible team, carrying the original requester’s details in the message body so the receiving team always knows who reported the problem. The interesting constraints were an ordered decision ladder rather than a flat category list (so mixed-domain emails resolve deterministically), a strict single-token output contract, and a fail-safe that sends anything ambiguous to a human.

**→ [Read the deep dive](docs/01-email-triage-routing.md)**

### 2. Service Desk Intelligence Agent

An autonomous teammate for the help desk. Everything it writes is for the technicians, on the team's side of the ticket. Every night it reads each open ticket the way a senior technician would, including screenshots, and leaves one pinned note: what's happening, what to do next, who it's waiting on.

The design problems worth reading about: a grounding hierarchy that prefers the desk's own past resolutions over vendor documentation and verifies every link it cites, per-capability blast radii that let a new behaviour prove itself on a single ticket before it touches the desk, and content fingerprinting so cost tracks how much actually changed rather than how many tickets are open.

**→ [Read the deep dive](docs/02-service-desk-intelligence-agent.md)**

### 3. Attack Surface & Exposure Management Platform

Continuous threat exposure management built on top of security tools the organization already licensed.

The architectural spine is that **deterministic code runs before the model.** Deduplication, enrichment, and tiering are ordinary code; the model runs last, sees a tier it cannot change, and contributes prose and ordering within it. Paging behavior can't shift because a vendor updated a model, and an auditor can reproduce a decision six months later.

*Write-up in progress.*

---

## How I Build These

The same principles recur across all three, and they're what I'd point to first.

**Deterministic where it matters, probabilistic where it helps.** A model is excellent at reading a wall of forwarded email and pulling out what actually matters. Where an answer needs to be the same tomorrow as it is today, ordinary code does that job better. So in each of these the model's authority is bounded by something a person can read in a diff — it classifies into a fixed set, or writes the prose around a decision it doesn't get to make.

**Fail toward a human, never toward a drop.** An ambiguous email routes to the service desk. A scope check that can't confirm ownership stops the scan. In each case the failure mode is a small cost paid visibly, instead of a quiet one paid later.

**Every write is gated and scoped.** Capabilities are separate switches with their own blast radii, so a new behaviour can prove itself on one ticket before it goes near the desk. The security platform escalates but never touches an asset, because in production an action you can't recall is its own incident.

**Untrusted content stays untrusted.** Ticket text, screenshots, resource names, and service banners are all attacker-reachable. They're passed as data, never concatenated into instructions, and a successful injection produces wrong prose rather than a wrong action.

**Cost proportional to change, not volume.** Unchanged work is skipped by fingerprint. Expensive analysis runs only on what reaches the top tier. One of these systems runs an entire routing function for about five dollars a month.

*The security platform's case study is still being written up.*

---

## About

Applied AI Engineer | Production LLM Systems, Agents & Automation

I design and ship reliable AI systems for real operational environments, with a focus on bounded autonomy, evaluation, observability, security, and measurable business outcomes.

My background in IT operations and cybersecurity informs how I build: failure modes, permissions, adversarial input, rollback, auditability, and production reliability are first-class design concerns.

Currently a Senior Security Administrator working across IT Operations & Security.

**[LinkedIn](https://linkedin.com/in/zanesolomon)**
