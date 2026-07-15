# stealth-proxy-runner

A lightweight headless browser automation server with built-in proxy rotation and anti-detection, built on top of [Camoufox](https://github.com/daijro/camoufox) — a Firefox fork with fingerprint spoofing at the C++ level.

## Why

AI agents and web automation scripts get blocked by Cloudflare, Google reCAPTCHA, and fingerprinting systems. This project combines stealth browser technology with flexible proxy rotation so you can browse the real web at scale.

**Core advantages:**
- C++-level fingerprint spoofing: `navigator.hardwareConcurrency`, WebGL renderers, AudioContext, screen geometry, WebRTC — all spoofed before JavaScript ever sees them
- Round-robin and residential backconnect proxy strategies with sticky sessions
- Works on $5 VPS, Raspberry Pi, or any cloud container — idle memory ~40MB
- REST API with token-efficient accessibility snapshots (~90% smaller than raw HTML)

## Features

- **C++ Anti-Detection** — bypasses Google, Cloudflare, and most bot detection systems
- **Proxy Rotation** — round-robin and backconnect (Decodo, BrightData, Oxylabs, generic) with session stickiness
- **Session Isolation** — separate cookies and storage per user/session
- **Proxy + GeoIP** — route through residential proxies with automatic locale/timezone sync
- **Element Refs** — stable `e1`, `e2`, `e3` identifiers for reliable interaction
- **Token-Efficient** — accessibility snapshots instead of bloated HTML
- **Deploy Anywhere** — Docker, Fly.io, Railway, bare metal
- **Structured Logging** — JSON log lines with request IDs for production observability

## Quick Start

### Install

```bash
git clone https://github.com/luoshixin93-sudo/stealth-proxy-runner
cd stealth-proxy-runner
npm install
```

### Run

```bash
npm start
# Server starts at http://localhost:9377
```

### Docker

```bash
docker run -d -p 9377:9377 \
  -e PROXY_HOST=your.proxy.com \
  -e PROXY_PORTS=10000,10001,10002 \
  -e PROXY_USER=your_username \
  -e PROXY_PASS=your_password \
  ghcr.io/luoshixin93-sudo/stealth-proxy-runner
```

## Proxy Configuration

### Round-Robin (static proxy pool)

```javascript
{
  strategy: 'round_robin',
  host: 'proxy.example.com',
  ports: [10000, 10001, 10002],
  username: 'user',
  password: 'pass'
}
```

### Backconnect (residential proxies)

```javascript
{
  strategy: 'backconnect',
  providerName: 'decodo',        // 'decodo' | 'generic'
  backconnectHost: 'zproxy.lum-superproxy.io',
  backconnectPort: 22225,
  username: 'your-username',
  password: 'your-password',
  country: 'US',
  sessionId: 'my-session-001', // optional, enables sticky sessions
  sessionDurationMinutes: 10     // optional, session TTL
}
```

## API Overview

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/tabs` | Open a new browser tab |
| `GET` | `/tabs/:tabId/snapshot` | Get accessibility snapshot with element refs |
| `POST` | `/tabs/:tabId/click` | Click element by ref or selector |
| `POST` | `/tabs/:tabId/type` | Type text into element |
| `POST` | `/tabs/:tabId/navigate` | Navigate to URL or search macro |
| `GET` | `/tabs/:tabId/screenshot` | Capture page screenshot |
| `GET` | `/tabs` | List all open tabs |
| `DELETE` | `/tabs/:tabId` | Close a tab |

Full OpenAPI spec available at [`/openapi.json`](http://localhost:9377/openapi.json).

## Example: Scrape Google Results with Proxy

```javascript
import fetch from 'node-fetch';

const BASE = 'http://localhost:9377';

// Open tab with proxy
const tab = await fetch(`${BASE}/tabs`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    proxy: {
      strategy: 'backconnect',
      providerName: 'decodo',
      backconnectHost: 'zproxy.lum-superproxy.io',
      backconnectPort: 22225,
      username: 'your-username',
      password: 'your-password',
      country: 'US'
    }
  })
}).then(r => r.json());

const tabId = tab.id;

// Navigate with search macro
await fetch(`${BASE}/tabs/${tabId}/navigate`, {
  method: 'POST',
  body: JSON.stringify({ url: '@google_search', query: 'site:github.com playwright' })
});

// Get snapshot
const snap = await fetch(`${BASE}/tabs/${tabId}/snapshot`).then(r => r.json());

// Click first result
await fetch(`${BASE}/tabs/${tabId}/click`, {
  method: 'POST',
  body: JSON.stringify({ ref: 'e1' })
});

console.log('Title:', snap.title);
console.log('URL:', snap.url);
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `9377` | Server port |
| `CAMOUFOX_EXECUTABLE` | auto | Path to Camoufox binary |
| `PROXY_STRATEGY` | `round_robin` | `round_robin` or `backconnect` |
| `PROXY_HOST` | — | Proxy host for round_robin |
| `PROXY_PORTS` | — | Comma-separated ports |
| `PROXY_USER` | — | Proxy username |
| `PROXY_PASS` | — | Proxy password |
| `CAMOFOX_CRASH_REPORT_ENABLED` | `true` | Anonymized crash telemetry |

## Credits

Standing on the shoulders of [Camoufox](https://github.com/daijro/camoufox) — a Firefox fork with fingerprint spoofing at the C++ level.

---

Made with ❤️ for cloud phone automation → [android-cloud-device.com](https://www.android-cloud-device.com/)
