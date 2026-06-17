# Scripting Guide

Outpost Beacon uses JavaScript check scripts to monitor services. Scripts are executed via Apple's built-in JavaScriptCore engine — no external runtime needed.

## Script Location

Scripts live in:

```
~/Library/Application Support/Outpost Beacon/checks/
```

On first launch, Outpost Beacon copies default check scripts into this directory.

## Script Metadata

Add comment directives at the top of your script to configure how Outpost Beacon handles it:

| Directive | Required | Description |
|-----------|----------|-------------|
| `// OUTPOST_NAME = "My Service"` | No | Display name (defaults to filename) |
| `// OUTPOST_URL = "https://..."` | No | Status page URL for the service |
| `// OUTPOST_INTERVAL = "60"` | No | Custom polling interval in seconds (minimum 30) |
| `// OUTPOST_ICON = "cloud.fill"` | No | SF Symbol name for the service icon (or use the icon picker in the editor) |
| `// OUTPOST_COLOR = "teal"` | No | Icon color: red, blue, green, orange, purple, teal, pink, indigo, mint, yellow, brown, cyan, gray |
| `// OUTPOST_GROUP = "Production"` | No | Group services together on the dashboard under a named, collapsible section |
| `// OUTPOST_DEPENDS = "GitHub, AWS"` | No | Comma-separated display names of services this one depends on; a degraded dependency flags this service as impacted |

## Available Functions

### HTTP Functions

| Function | Returns | Description |
|----------|---------|-------------|
| `fetch(url)` | JSON object | HTTP GET, parses response as JSON |
| `fetch(url, {encoding: "utf-16"})` | JSON object | HTTP GET with custom response encoding |
| `fetchResponse(url)` | `{status, body}` | HTTP GET with status code access |
| `fetchText(url)` | String | HTTP GET, returns raw text |
| `fetchAll([url1, url2, ...])` | Array | Concurrent HTTP GET, returns array of parsed JSON |
| `fetchHeaders(url)` | `{success, statusCode, headers, error}` | Inspect HTTP response headers (HEAD by default; pass `{method: "GET"}` to use GET). Header names are lowercased. |
| `fetchAndHash(url)` | `{success, statusCode, hash, byteCount, error}` | SHA-256 hash of the response body, for detecting content changes |
| `bodyContains(url, expected)` | `{success, statusCode, contains, error}` | Whether the response body contains a substring (case-sensitive) |
| `fetchRequest(url, options)` | `{success, status, ok, headers, body, json, error}` | Send a request with a custom `method`, `headers`, and `body` (GET/POST/PUT/PATCH/DELETE). A string body is sent as-is; an object body is JSON-encoded with a default `Content-Type` unless you set one. |

### Check Helpers

| Function | Returns | Description |
|----------|---------|-------------|
| `statuspageCheck(url)` | — | One-liner for Statuspage.io services (calls `output()` for you) |
| `statuspageCheck(url, {showcaseFilter: false})` | — | Same, but includes all components |
| `tcpCheck(host, port)` | `{success, latencyMs, error}` | TCP connection check |
| `tcpCheck(host, port, {timeout: 5})` | `{success, latencyMs, error}` | TCP check with custom timeout (seconds) |
| `certCheck(host)` | `{valid, trusted, daysRemaining, notAfter, issuer, sanList, ...}` | Inspect a TLS certificate's expiry and trust (use `certCheck(host, port)` for a non-443 port) |
| `dnsLookup(hostname)` | `{success, records, server, ...}` | DNS query (system resolver), A records by default |
| `dnsLookup(hostname, "MX")` | `{success, records, ...}` | Query a specific record type: A, AAAA, CNAME, MX, TXT, NS |
| `dnsLookup(hostname, "A", {server: "10.0.0.1"})` | `{success, records, ...}` | Query a specific DNS server directly |
| `domainExpiry(domain)` | `{success, expiryDate, daysRemaining, registrar, ...}` | Domain registration expiry via RDAP |
| `whoisQuery(domain, server)` | `{success, raw, expiryDate, daysRemaining, error}` | Raw WHOIS lookup for TLDs without RDAP |

### Utility Functions

| Function | Description |
|----------|-------------|
| `output(obj)` | Set the script's result **(required, call exactly once)** |
| `stripHtml(text)` | Remove HTML tags and decode entities |
| `log(message)` | Debug logging (visible in Console.app and the script editor console) |

## Status Values

Use these strings for `status`, component `status`, and incident `impact` fields:

| Value | Meaning |
|-------|---------|
| `operational` | All systems normal |
| `degraded` | Degraded performance |
| `partial_outage` | Partial outage |
| `major_outage` | Major outage |

## Output Schema

Only `status` is required. Everything else is optional:

```json
{
  "status": "operational",
  "responseTimeMs": 142,
  "components": [
    { "name": "API", "status": "operational", "description": "Optional detail" }
  ],
  "incidents": [
    {
      "title": "Elevated error rates",
      "status": "Investigating",
      "impact": "minor",
      "created_at": "2025-01-15T10:00:00Z",
      "is_active": true,
      "updates": [
        { "body": "Looking into it.", "status": "Investigating", "created_at": "2025-01-15T10:05:00Z" }
      ]
    }
  ]
}
```

## Examples

### Statuspage.io Service

```javascript
// OUTPOST_NAME = "GitHub"
// OUTPOST_URL = "https://www.githubstatus.com"

statuspageCheck("https://www.githubstatus.com/api/v2/summary.json");
```

### Simple Health Check

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

### HTTP Status Code Check

```javascript
// OUTPOST_NAME = "My API"
// OUTPOST_URL = "https://api.example.com"

var res = fetchResponse("https://api.example.com/health");

if (res.status === 503) {
    output({ status: "major_outage" });
} else if (res.status === 429 || res.status >= 500) {
    output({ status: "degraded" });
} else if (res.status === 200) {
    output({ status: "operational" });
} else {
    output({ status: "unknown" });
}
```

### POST Request (GraphQL or JSON API)

Use `fetchRequest()` when a check needs a method and body — for example, a GraphQL
health query or a JSON-RPC call. The result is structured, so you branch on
`result.success` and `result.ok` instead of using try/catch.

```javascript
// OUTPOST_NAME = "GraphQL API"
// OUTPOST_URL = "https://api.example.com"

var res = fetchRequest("https://api.example.com/graphql", {
    method: "POST",
    headers: { "Authorization": "Bearer " + "TOKEN" },
    body: { query: "{ health { status } }" }   // object body is JSON-encoded
});

if (!res.success || !res.ok) {
    output({ status: "major_outage" });
} else if (res.json && res.json.data && res.json.data.health.status === "ok") {
    output({ status: "operational" });
} else {
    output({ status: "degraded" });
}
```

### TCP Port Check

```javascript
// OUTPOST_NAME = "Database Cluster"

var primary = tcpCheck("db-primary.example.com", 5432, { timeout: 3 });
var replica = tcpCheck("db-replica.example.com", 5432, { timeout: 3 });

var components = [
    {
        name: "Primary (db-primary:5432)",
        status: primary.success ? "operational" : "major_outage",
        description: primary.success ? "Latency: " + primary.latencyMs + "ms" : primary.error
    },
    {
        name: "Replica (db-replica:5432)",
        status: replica.success ? "operational" : "major_outage",
        description: replica.success ? "Latency: " + replica.latencyMs + "ms" : replica.error
    }
];

var status = "operational";
if (!primary.success) status = "major_outage";
else if (!replica.success) status = "partial_outage";

output({ status: status, components: components, responseTimeMs: primary.latencyMs });
```

### Custom API with Components

```javascript
// OUTPOST_NAME = "Internal Platform"
// OUTPOST_URL = "https://status.internal.example.com"

var data = fetch("https://status.internal.example.com/api/health");

var components = (data.services || []).map(function(svc) {
    return {
        name: svc.name,
        status: svc.healthy ? "operational" : "major_outage",
        description: svc.message || null
    };
});

var worst = "operational";
components.forEach(function(c) {
    if (c.status === "major_outage") worst = "major_outage";
    else if (c.status === "degraded" && worst === "operational") worst = "degraded";
});

output({ status: worst, components: components });
```

## Tips

- **Disable a check** — remove the `.js` file or rename its extension (e.g., `.js.disabled`)
- **Reorder checks** — rename files with numeric prefixes (`01-`, `02-`, etc.)
- **Debug a script** — use the built-in script editor's Run button, or use `log()` and check Console.app
- **Reload after edits** — go to Settings → Scripts → Reload Scripts, or restart the app
- **Script errors** — if a script throws or never calls `output()`, the service shows as "Unknown"
