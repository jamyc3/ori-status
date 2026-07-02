# ori-status

External uptime monitor for **Ori** and **Jot** production surfaces, running on GitHub
Actions every ~5 minutes. No third-party service, no account, no secrets — it only
probes public URLs.

## What it checks
| Check | URL | Passes when |
|---|---|---|
| Ori app | `talk-to-me.ideaflow.page/` | 200 + SPA bundle (`index-*.js`) served |
| Ori API health | `talk-to-me.ideaflow.page/proxy/health` | 200 + `status` is `ok` or `degraded` |
| Ori marketing site | `ori-app.ideaflow.page/` | 200 |
| Jot website | `jot-transcribe.com/` | 200 |

Each check retries 3× (10 s apart) before failing, so a single blip doesn't page anyone.

## How you get alerted
1. **Email from GitHub** — a failing run sends the standard "Run failed: uptime" email
   to the repo owner's GitHub notification address. Recovery = runs go green (no email).
2. **Incident issue** — the first failing run opens a `🔴 Ori/Jot production incident`
   issue (subsequent failures comment on it; recovery closes it automatically). Anyone
   **watching this repo** gets issue notifications — so a second person/address can
   subscribe by watching the repo from their GitHub account.

## Test the alert path
Actions → `uptime` → Run workflow → set `force_fail` = `true`. You'll get the failure
email + a test incident issue (auto-closed by the next green run).

## Why this exists
The in-app watchdog writes Gmail *drafts* (its connector can't send), and its cloud
sandbox can be egress-blocked. This monitor lives outside all of that infrastructure
and pushes real notifications by itself. Detected outage 2026-07-02 (~13:00–14:00 UTC,
gateway 403 on everything) motivated it.
