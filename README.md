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

## Documentation

Full documentation lives in the **[Outpost Beacon Wiki](https://github.com/mcherry/Outpost-Beacon/wiki)**:

- **[Quick Start](https://github.com/mcherry/Outpost-Beacon/wiki/Quick-Start)** — install and create your first check
- **[Scripting Guide](https://github.com/mcherry/Outpost-Beacon/wiki/Scripting-Guide)** — how check scripts work, with examples
- **[Metadata Reference](https://github.com/mcherry/Outpost-Beacon/wiki/Metadata-Reference)** — all script metadata directives
- **[Editor Guide](https://github.com/mcherry/Outpost-Beacon/wiki/Editor-Guide)** — using the built-in script editor
- **[Troubleshooting](https://github.com/mcherry/Outpost-Beacon/wiki/Troubleshooting)** and **[FAQ](https://github.com/mcherry/Outpost-Beacon/wiki/FAQ)**

## Custom Status Checks

Outpost Beacon runs JavaScript check scripts using Apple's built-in JavaScriptCore engine — no external runtime required. Scripts live in:

```
~/Library/Application Support/Outpost Beacon/checks/
```

A check script reads whatever it needs, decides a status, and calls `output()` once:

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

Scripts can check HTTP endpoints, TCP ports, TLS certificates, DNS records, and domain expiry, and can group services and declare dependencies. See the **[Scripting Guide](https://github.com/mcherry/Outpost-Beacon/wiki/Scripting-Guide)** and **[Metadata Reference](https://github.com/mcherry/Outpost-Beacon/wiki/Metadata-Reference)** for the full API, every metadata directive, and more examples.

## Support

- **Support page** — [Outpost Beacon Support](https://mcherry.github.io/Outpost-Beacon/support.html)
- **Bug reports & feature requests** — [open an issue](https://github.com/mcherry/Outpost-Beacon/issues)
- **Email** — info@inditech.org

## Privacy

Outpost Beacon does not collect, transmit, or share any personal data. See the full [Privacy Policy](https://mcherry.github.io/Outpost-Beacon/privacy.html).

## License

Copyright © 2025 Mike Cherry. All rights reserved.
