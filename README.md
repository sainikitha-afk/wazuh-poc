# Wazuh SIEM/SOAR Proof of Concept

A hands-on build of a working Wazuh SIEM deployment in an isolated AWS sandbox —
demonstrating log ingestion, detection engineering, alerting, and automated
response (SOAR) against a deliberately vulnerable target application.

## What this actually is (plain-English)

This is **not** one machine doing everything. It's **two separate virtual
machines, both running in AWS's cloud** (not on any local laptop), each doing
one job, talking to each other over a private network:

- **`wazuh-manager`** — a security office. Runs Docker, and inside Docker,
  three containers work together: one watches for suspicious patterns (the
  actual "manager"), one stores every event ever reported (the "indexer"),
  and one is the web dashboard a human logs into to look at everything.
- **`linux-worker`** — a decoy shop with intentionally broken locks. Runs
  DVWA (Damn Vulnerable Web Application) — a real, deliberately vulnerable
  website — plus a small background program (the Wazuh **agent**) that
  watches everything happening on this machine and reports back to
  `wazuh-manager` over the network. No Docker here at all — DVWA and the
  agent are both installed the plain, ordinary way.

A laptop is only ever the remote control — it opens an SSH connection to
each machine to run commands, but nothing described above actually runs on
the laptop itself.

## Architecture
                AWS VPC (isolated private network)
    ┌─────────────────────────────────────────────────────┐
    │                                                       │
    │   linux-worker (EC2)              wazuh-manager (EC2) │
    │   ┌─────────────────┐             ┌──────────────────┐│
    │   │ Apache + PHP     │             │  Docker          ││
    │   │ + MySQL + DVWA   │             │  ┌────────────┐  ││
    │   │                  │  reports    │  │  manager   │  ││
    │   │ Wazuh agent  ────┼────────────►│  ├────────────┤  ││
    │   │ (no Docker)      │  port 1514/ │  │  indexer   │  ││
    │   └─────────────────┘  1515        │  ├────────────┤  ││
    │                                    │  │  dashboard │  ││
    │                                    │  └────────────┘  ││
    │                                    └──────────────────┘│
    └─────────────────────────────────────────────────────┘
                                                 │
                                                 ▼
                                      Alert → Slack / email
                                                 │
                                                 ▼
                                Active Response → auto-block attacker IP


## Why each piece exists

| Component | Purpose |
|---|---|
| AWS VPC + subnets | An isolated private network so these machines aren't exposed to the whole internet by default |
| Security Groups | Per-machine firewalls — only specific ports, from specific sources, are allowed in |
| `wazuh-manager` (EC2) | Hosts the full Wazuh "brain" stack via Docker |
| Docker (on manager only) | Packages manager/indexer/dashboard as pre-built, version-matched containers instead of installing each by hand |
| `linux-worker` (EC2) | The target machine — runs the thing we're going to attack (DVWA) and the agent that reports on it |
| DVWA | A real, intentionally vulnerable web app — something legitimate to practice attacking in a legal, contained way |
| Wazuh agent | A lightweight always-on program that watches logs on `linux-worker` and ships them to the manager |
| Detection rules (XML) | Teach the manager what "suspicious" actually looks like — without these, the manager just stores data with no opinion on it |
| Alerting (Slack/email) | Routes high-severity detections to a human, instead of requiring someone to babysit the dashboard |
| Active Response | Automated containment — the manager can act (e.g. block an IP) without a human doing it manually |
| Dashboard visualizations | Turns raw stored events into an at-a-glance operational view |

## Status

| Phase | What | Status |
|---|---|---|
| 0 | AWS infra verified (VPC, subnets, security groups, key pair) | ✅ Done |
| 1 | EC2 instances launched (`wazuh-manager`, `linux-worker`) | ✅ Done |
| 2 | Wazuh stack (manager/indexer/dashboard) deployed via Docker, verified healthy | ✅ Done |
| 3 | DVWA + LAMP stack deployed on `linux-worker` | ✅ Done |
| 4 | Wazuh agent installed on `linux-worker`, confirmed **Active** in dashboard | ✅ Done |
| 5 | Detection rules: brute-force, SQL injection, correlation (MITRE ATT&CK tagged) | ⬜ In progress |
| 6 | Alerting (Slack + email) on high-severity matches | ⬜ Not started |
| 7 | Active Response — auto-block attacker IP | ⬜ Not started |
| 8 | Dashboard: agent status, severity breakdown, custom detections | ⬜ Not started |
| — | `windows-worker` (agent-only, lowest priority) | ⬜ Not started |

## Tech stack

AWS EC2 / VPC / Security Groups · Docker + Docker Compose · Wazuh 4.14.7
(manager, indexer, dashboard) · DVWA · Ubuntu 24.04 LTS · Apache · MySQL · PHP 8.3
