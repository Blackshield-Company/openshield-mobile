# OpenShield Pocket — design brief

Excerpted from the OpenShield Desktop design (source of truth:
`openshield-desktop/docs/openshield-desktop-design.md`).

## What Pocket is

**OpenShield Pocket** is the mobile port (same codebase) that pairs with a
desktop `openshield serve` instance over LAN. The operator can watch runs,
approve risky tool calls, and send chat from the phone. Strictly a remote —
no offline provider chat.

## Stack

- Same SvelteKit frontend + Rust core as desktop, platform-gated.
- Android target: Tauri mobile (`src-tauri/gen/android`).
- App ID: `com.openshield.pocket`.
- Transport: WebSocket (not Tauri IPC) — the same code path works on desktop
  and Android, so chat, tool cards, and approval flows behave identically.

## Pairing (M4)

1. Desktop runs `openshield serve` 1.1.1 or newer. Token required on every
   request, including loopback.
2. Pocket discovers or is pointed at the host:port.
3. Pair via `POST /api/v1/pair` — QR scan or manual paste of the pairing
   payload (host, port, token, expiry). Payloads are minted with the master
   token on the desktop; only the master token may call pair.
4. Token stored app-private (Tauri keyring/stronghold), never shared storage.
   The app always sends the bearer token; WS uses `?token=`.
5. A 401 shows a re-pair banner ("Unauthorized — re-pair with your OpenShield
   server") and routes to Settings until re-paired. A 429 surfaces "Too many
   pairing attempts, wait a minute".

## Security model

- Token required on every request, including loopback. Minimum server:
  openshield 1.1.1.
- Wildcard (`0.0.0.0`) binds refused unless `lan_mode = true` — explicit
  opt-in for mobile access.
- Default: `bind 127.0.0.1`, port 9310, `lan_mode false`.
- Pairing payloads expire and are minted with the master token on the
  desktop. Tokens are per-session, revocable via the desktop. `POST
  /api/v1/pair` is master-token only and rate-limited to 5/min (429).
- Mobile approval gating mirrors desktop: WS `approval_req/res` with a 120s
  deny timeout; `/api/v1/approvals` REST for the inbox.

## Roadmap

- [x] M4 — Android project, manual/QR pairing, app-private token storage,
      first-run routing (code done; on-device pass pending)
- [ ] On-device install + screencap pass on the Android target
- [ ] v0.1 release packaging

The shield holds. ⚫🛡