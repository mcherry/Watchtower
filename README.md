# Outpost Beacon

A macOS menu bar app that monitors the health of key infrastructure services at a glance.

<p align="center">
  <img src="screenshots/dashboard.png" width="800" alt="Status dashboard — all services at a glance">
</p>

## Features

- **Menu bar status icon** — changes color and shape to reflect overall service health
- **Status dashboard** — floating panel shows all monitored services with component-level detail
- **Service groups** — organize services into named, collapsible groups; the dashboard scales to large service lists with a pinned at-a-glance health summary
- **Resizable dashboard** — drag to resize the panel; it remembers its size and always reappears centered under the menu bar icon
- **Dashboard pinning** — pin the dashboard to keep it visible when clicking outside
- **Service drill-down** — click any service to see components, active incidents, and recent event history
- **Service dependencies** — declare what a service relies on and see at a glance when a degraded dependency puts it at risk
- **Downtime duration** — non-operational services show how long they've been in their current state
- **Global keyboard shortcut** — configurable hotkey to toggle the dashboard
- **Launch at Login** — optional automatic startup when you log in
- **Background polling** — refreshes service status on a configurable interval, with optional per-script intervals
- **macOS notifications** — opt-in alerts when a service degrades or recovers
- **Quiet hours** — silence notifications during a daily time window, such as overnight, with an option to also pause checks
- **Response time tracking** — measures and displays service response time
- **Response time alerts** — set per-service response time thresholds and get notified when a service gets too slow
- **Webhook & automation** — trigger webhooks, Slack, or Discord messages on status changes and response time breaches, with self-signed certificate support for internal services
- **Uptime history** — opt-in graphical timeline showing historical uptime and response times per service
- **Incident timeline** — a chronological feed of service status changes with how long each state lasted, grouped by day
- **Configurable data retention** — choose how long history is kept, see the current database size, and clear monitoring history on demand
- **Uptime report export** — export uptime data as CSV or PDF
- **Custom status checks** — write JavaScript scripts to monitor any service using the built-in API
- **AI script assistant** (Beta) — describe a check in plain language and have a starting script drafted for you, either fully on-device with Apple Intelligence or via your own local/remote AI server; you review and edit it before it's used
- **Built-in script editor** — write and debug check scripts with syntax highlighting, code folding, inline validation warnings, auto-completion, hover API docs, a network request log, templates, and a run console
- **Dark mode support** — adapts to system appearance

<p align="center">
  <img src="screenshots/script-editor.png" width="800" alt="Built-in script editor with validation, completion, and the network request log">
</p>

<p align="center">
  <img src="screenshots/uptime-history.png" width="800" alt="Uptime history with health timeline and response-time charts">
</p>

<p align="center">
  <img src="screenshots/settings-general.png" width="410" alt="Settings — General">
  &nbsp;&nbsp;
  <img src="screenshots/settings-scripts.png" width="410" alt="Settings — Scripts">
</p>

<p align="center">
  <img src="screenshots/settings-alerts.png" width="410" alt="Settings — Response Time Alerts">
  &nbsp;&nbsp;
  <img src="screenshots/settings-automations.png" width="410" alt="Settings — Automations">
</p>

## Requirements

- macOS 14.0 (Sonoma) or later

## Installation

Outpost Beacon is available on the [Mac App Store](https://apps.apple.com/app/outpost-beacon/id6772313616).

## Custom Status Checks

Outpost Beacon runs JavaScript check scripts using Apple's built-in JavaScriptCore engine — no external runtime required. Scripts live in:

```
~/Library/Application Support/Outpost Beacon/checks/
```

### Script Metadata

| Comment Directive | Required | Description |
|-------------------|----------|-------------|
| `// OUTPOST_NAME = "My Service"` | No | Display name (defaults to filename without extension) |
| `// OUTPOST_URL = "https://..."` | No | Status page URL for the service |
| `// OUTPOST_INTERVAL = "60"` | No | Custom polling interval in seconds (minimum 30) |
| `// OUTPOST_ICON = "cloud.fill"` | No | SF Symbol name for the service icon in the dashboard. Use the icon picker in the script editor or any [SF Symbol name](https://developer.apple.com/sf-symbols/). |
| `// OUTPOST_COLOR = "teal"` | No | Icon color. Options: red, blue, green, orange, purple, teal, pink, indigo, mint, yellow, brown, cyan, gray. |
| `// OUTPOST_GROUP = "Production"` | No | Group services together on the dashboard under a named, collapsible section. Also assignable from Settings → Scripts. |
| `// OUTPOST_DEPENDS = "GitHub, AWS"` | No | Comma-separated display names of services this one depends on. When a dependency is degraded, this service is flagged as impacted. |

### Available Functions

| Function | Description |
|----------|-------------|
| `fetch(url)` | HTTP GET → parsed JSON object |
| `fetch(url, {insecure: true})` | HTTP GET accepting self-signed certificates |
| `fetchResponse(url)` | HTTP GET → `{ status, body }` |
| `fetchResponse(url, {insecure: true})` | Same, accepting self-signed certificates |
| `fetchText(url)` | HTTP GET → raw response string |
| `fetchText(url, {insecure: true})` | Same, accepting self-signed certificates |
| `fetchAll([url1, url2, ...])` | Concurrent HTTP GET → array of parsed JSON |
| `fetchHeaders(url)` | Inspect HTTP response headers → `{ success, statusCode, headers, error }` |
| `fetchAndHash(url)` | SHA-256 hash of the response body (content-drift detection) → `{ success, statusCode, hash, byteCount, error }` |
| `bodyContains(url, expected)` | Check whether a response body contains a substring → `{ success, statusCode, contains, error }` |
| `output(obj)` | Set the script's result (required, call once) |
| `statuspageCheck(url)` | One-liner for any Statuspage.io service |
| `tcpCheck(host, port, options?)` | TCP connect check → `{ success, latencyMs, error }` |
| `certCheck(host[, port])` | Inspect a TLS certificate's expiry and trust → `{ valid, trusted, daysRemaining, notAfter, issuer, ... }` |
| `dnsLookup(hostname[, type][, options])` | DNS query with TTLs (A/AAAA/CNAME/MX/TXT/NS); optionally query a specific resolver |
| `domainExpiry(domain)` | Domain registration expiry via RDAP → `{ success, expiryDate, daysRemaining, registrar, ... }` |
| `whoisQuery(domain, server)` | Raw WHOIS lookup for TLDs without RDAP → `{ success, raw, expiryDate, daysRemaining, error }` |
| `stripHtml(text)` | Remove HTML tags and decode entities |
| `log(message)` | Debug logging |

### Example: Statuspage.io Service

```javascript
// OUTPOST_NAME = "GitHub"
// OUTPOST_URL = "https://www.githubstatus.com"

statuspageCheck("https://www.githubstatus.com/api/v2/summary.json");
```

### Example: HTTP Health Check

```javascript
// OUTPOST_NAME = "My API"
// OUTPOST_URL = "https://api.example.com"

try {
    fetch("https://api.example.com/health");
    output({ status: "operational" });
} catch (e) {
    output({ status: "major_outage" });
}
```

### Example: TCP Port Check

```javascript
// OUTPOST_NAME = "Database"


var result = tcpCheck("db.example.com", 5432, { timeout: 3 });
output({
    status: result.success ? "operational" : "major_outage",
    responseTimeMs: result.latencyMs
});
```

### Example: Self-Signed Certificate

Pass `{insecure: true}` to accept self-signed or untrusted certificates — useful for internal services, NAS devices, and development servers:

```javascript
// OUTPOST_NAME = "My NAS"
// OUTPOST_URL = "https://nas.local:8443"

var data = fetch("https://nas.local:8443/api/health", {insecure: true});
output({ status: data.ok ? "operational" : "major_outage" });
```

### Example: SSL Certificate Expiry

Warn before a TLS certificate lapses:

```javascript
// OUTPOST_NAME = "example.com TLS"

var c = certCheck("example.com");
if (!c.valid || c.daysRemaining <= 7) {
    output({ status: "major_outage" });
} else if (c.daysRemaining <= 30) {
    output({ status: "degraded" });
} else {
    output({ status: "operational" });
}
```

For more examples and the full scripting reference, see the [Scripting Guide](docs/scripting.md).

## Support

- **Support page** — [Outpost Beacon Support](https://mcherry.github.io/Outpost-Beacon/support.html)
- **Bug reports & feature requests** — [open an issue](https://github.com/mcherry/Outpost-Beacon/issues)
- **Email** — info@inditech.org

## Privacy

Outpost Beacon does not collect, transmit, or share any personal data. See the full [Privacy Policy](https://mcherry.github.io/Outpost-Beacon/privacy.html).

## License

Copyright © 2025 Mike Cherry. All rights reserved.
