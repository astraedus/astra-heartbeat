# astra-heartbeat — off-box dead-man's-switch

Guards the Astraedus autonomy engine against **silent full-box blackouts**.

## Why
On 2026-08-11..19 the laptop was host-suspended ~8 days. Every "Telegram on
failure" cron alert is sent *from the box* — so when the box itself is dark,
those alerts never send and nobody knows the machine is dead (it only came back
by coincidence). On-box alerts have a blind spot exactly when it matters most.
See `~/ops/LESSONS.md` 2026-08-19.

## How it works
- **On-box** (`~/bin/astra-heartbeat.sh`, hourly cron): writes the current epoch
  to `heartbeat.txt` and pushes it here.
- **Off-box** (`.github/workflows/heartbeat-check.yml`, GitHub Actions cron,
  hourly): runs on GitHub's infra. If `heartbeat.txt` is older than 3h it sends
  a Telegram alert to Anti **from the GitHub runner** — which survives even a
  total box outage. Throttled to ≤1 repeat alert / 6h during an ongoing outage.

Complements `astra-auth-health.sh` (catches OAuth-expiry) — this catches the
box-is-dead / suspended case that auth-health can't (it also runs on the box).

## Manual controls
- Drill (prove the Telegram leg): `gh workflow run heartbeat-check.yml -f drill=true`
- Dry-run stale logic (no real send): `gh workflow run heartbeat-check.yml -f dry_run=true`

Secrets required (repo settings): `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`.
