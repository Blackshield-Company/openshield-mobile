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

1. Desktop runs `openshield serve` (LAN mode, bearer token).
2. Pocket discovers or is pointed at the host:port.
3. Pair via `POST /api/v1/pair` — QR scan or manual paste of the pairing
   payload (host, port, token, expiry).
4. Token stored app-private (Tauri keyring/stronghold), never shared storage.
5. Unpaired requests are rejected with 401 — pocket routes first-run to
   Settings until a session exists.

## Security model

- Wildcard (`0.0.0.0`) binds refused unless `lan_mode = true` — explicit
  opt-in for mobile access.
- Default: `bind 127.0.0.1`, port 9310, `lan_mode false`.
- Pairing payloads expire; tokens are per-session, revocable via the desktop.
- Mobile approval gating mirrors desktop: WS `approval_req/res` with a 120s
  deny timeout; `/api/v1/approvals` REST for the inbox.

## Roadmap

- [x] M4 — Android project, manual/QR pairing, app-private token storage,
      first-run routing (code done; on-device pass pending)
- [ ] On-device install + screencap pass on the Android target
- [ ] v0.1 release packaging

The shield holds. ⚫🛡