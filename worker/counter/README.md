# tesla-counter — anonymous usage counter

Cloudflare Worker that receives once-per-day ping from firmware and exposes
aggregate counts. No IP, no MAC, no serial — only `sha256(MAC)` hex truncated to
16 chars + firmware version + env tag.

## Deploy

```sh
cd worker/counter
npx wrangler deploy
```

After first deploy the URL looks like:
`https://tesla-counter.<your-subdomain>.workers.dev`

Paste that URL into `include/mod_telemetry_ping.h` constant `kPingEndpoint`
(stripping the trailing slash) and rebuild firmware.

## Verify

```sh
# Ping (should return "ok" or "ok (dedup)" if already counted today)
curl -X POST https://tesla-counter.<sub>.workers.dev/ping \
  -H 'Content-Type: application/json' \
  -d '{"id":"aabbccddeeff0011","version":"1.4.23","env":"esp32_wifi"}'

# Stats
curl https://tesla-counter.<sub>.workers.dev/stats
```

## Privacy

- Device ID is `sha256(MAC)` hex truncated to 16 hex chars (64 bits). Not
  reversible to MAC.
- No IP is stored. `cf-ipcountry` header (ISO country code, populated by
  Cloudflare edge) is stored per-device-per-day.
- KV TTL = 3 days. Stats endpoint shows today + yesterday rolling window.
- Firmware side has an opt-out toggle (NVS `ub_ping`). When disabled the
  firmware never contacts this worker.
