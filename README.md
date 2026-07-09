# ori-status

External uptime monitor for **Ori** and **Jot** production surfaces, running on GitHub
Actions. No third-party service, no account, no secrets — it only probes public URLs.

**How often it really runs:** GitHub throttles scheduled workflows heavily on free
public repos — in practice the schedule fires every ~1–4 hours, not every 5 minutes.
To compensate, each fired run probes in a **rolling window**: one probe cycle every
~5 minutes for up to ~45 minutes, and a newly fired run cancels the previous one
(cancelled runs in the history are normal takeovers, not problems). Failures alert
immediately mid-window. Net effect: real 5-minute granularity *while a window is
open*, with blind gaps between windows when GitHub delays the schedule. For true
always-on 5-minute coverage, add an external pinger (e.g. UptimeRobot free tier, or
a cron on the VPS) and keep this as the second opinion.

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
Actions → `uptime` → Run workflow → set `force_fail` = `true` (and `cycles` = `1` so
the test finishes fast). You'll get the failure email + a test incident issue
(auto-closed by the next green run).

## Infra flakes
GitHub occasionally fails a run before any step executes ("job was not acquired by
runner"). `rerun-flakes.yml` detects that signature (zero steps executed) and retries
the run once — real probe failures are never auto-retried. The "Run failed" email for
the flaky attempt still arrives (GitHub sends it before the retry can happen); if the
run is green when you click through, it was a flake.

## Why this exists
The in-app watchdog writes Gmail *drafts* (its connector can't send), and its cloud
sandbox can be egress-blocked. This monitor lives outside all of that infrastructure
and pushes real notifications by itself. Detected outage 2026-07-02 (~13:00–14:00 UTC,
gateway 403 on everything) motivated it.
