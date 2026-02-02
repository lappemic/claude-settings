---
name: umami-cloud
description: "Configure Umami Cloud analytics via its internal API using browser automation. Use when the user asks to set up, create, or manage Umami Cloud segments, cohorts, goals, funnels, journeys, retention reports, UTM reports, attribution, breakdown, or revenue reports. Also use when the user says 'set up analytics', 'create funnels', 'add goals in Umami', 'configure Umami', or wants to batch-create analytics configurations in Umami Cloud. Requires the claude-in-chrome MCP tools with an authenticated Umami Cloud session open in the browser."
---

# Umami Cloud API Configuration

Configure Umami Cloud analytics by calling its internal API via `mcp__claude-in-chrome__javascript_tool`, bypassing modal dialogs that block the Chrome extension.

## Prerequisites

1. User must be logged into Umami Cloud at `cloud.umami.is` in Chrome
2. The `claude-in-chrome` MCP tools must be available
3. Get tab context with `tabs_context_mcp` first

## Setup: Detect Region, Website ID, and Auth

Navigate to or identify the Umami tab, then extract config:

```js
// Execute via mcp__claude-in-chrome__javascript_tool
(async () => {
  const token = localStorage.getItem('umami.auth'); // raw string, not JSON
  const url = window.location.href;
  const regionMatch = url.match(/analytics\/(\w+)\//);
  const websiteMatch = url.match(/websites\/([a-f0-9-]+)/);
  return JSON.stringify({
    region: regionMatch?.[1],
    websiteId: websiteMatch?.[1],
    hasToken: !!token
  });
})();
```

Base URL pattern: `https://cloud.umami.is/analytics/{region}/api`

## API Reference

All calls require headers:
```js
{ 'Authorization': `Bearer ${token}`, 'Content-Type': 'application/json' }
```

### Segments & Cohorts

Same endpoint, differentiated by `type: 'segment'` or `type: 'cohort'`.

```
POST   /api/websites/{websiteId}/segments
GET    /api/websites/{websiteId}/segments?type=segment|cohort
DELETE /api/websites/{websiteId}/segments/{id}
```

Body:
```js
{ websiteId, name, type: 'segment'|'cohort', parameters: { filters: [{ type, value }] } }
```

Filter types: `device`, `language`, `referrer`, `browser`, `os`, `country`, `region`, `city`, `path`, `query`, `title`, `hostname`, `tag`, `event`

### Reports

All report types share one endpoint. Valid types: `goal`, `funnel`, `journey`, `retention`, `utm`, `attribution`, `breakdown`, `revenue`.

```
POST   /api/reports
GET    /api/reports?websiteId={id}
DELETE /api/reports/{id}
```

Base body: `{ websiteId, name, type, parameters }`

**Goal** — track event conversion rates:
```js
parameters: { startDate, endDate, type: 'event', value: 'event_name' }
// Optional: operator ('count'|'sum'|'average'), property (string)
```

**Funnel** — multi-step conversion paths (2-8 steps):
```js
parameters: { startDate, endDate, window: 7, steps: [{ type: 'path'|'event', value }] }
```

**Journey** — user path visualization (2-7 steps):
```js
parameters: { startDate, endDate, steps: 5, startStep: '/path', endStep: '' }
```

**Retention** — cohort return rates:
```js
parameters: { startDate, endDate, timezone: 'Europe/Zurich' } // timezone optional
```

**UTM** — campaign tracking:
```js
parameters: { startDate, endDate }
```

**Attribution** — conversion attribution:
```js
parameters: { startDate, endDate, model: 'first-click'|'last-click', type: 'path'|'event', step: 'event_name', currency: 'CHF' }
```

**Breakdown** — field-based breakdown:
```js
parameters: { startDate, endDate, fields: ['device', 'browser', 'country'] }
// Valid fields: path, referrer, title, query, os, browser, device, country, region, city, tag, hostname, language, event
```

**Revenue** — revenue tracking:
```js
parameters: { startDate, endDate, timezone: 'Europe/Zurich', currency: 'CHF' }
```

## Workflow

### 1. Get tab and auth context

```
tabs_context_mcp -> identify Umami tab
javascript_tool -> extract region, websiteId, verify token
```

### 2. Batch-create items

Use async IIFE pattern in `javascript_tool`:

```js
(async () => {
  const token = localStorage.getItem('umami.auth');
  const baseUrl = 'https://cloud.umami.is/analytics/{region}/api';
  const websiteId = '{websiteId}';
  const endDate = new Date().toISOString();
  const startDate = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString();

  const items = [ /* array of items to create */ ];
  const results = [];
  for (const item of items) {
    const res = await fetch(`${baseUrl}/reports`, {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${token}`, 'Content-Type': 'application/json' },
      body: JSON.stringify({ websiteId, name: item.name, type: item.type, parameters: item.parameters })
    });
    const data = await res.json();
    results.push({ name: item.name, status: res.status, id: data.id });
  }
  return JSON.stringify(results);
})();
```

### 3. Verify

Navigate to the relevant Umami page and take a screenshot to confirm items render correctly.

## Key Gotchas

- **Auth token** is a raw string in localStorage, not JSON — do not `JSON.parse()` it
- **Goal parameters** require `startDate`/`endDate` as ISO strings, plus `type` and `value` at the top level (not nested in a `goals` array) — without dates, the UI shows "Something went wrong"
- **Funnel steps** need min 2, max 8 entries; `window` is the conversion window in days
- **Journey steps** is the number of path columns (2-7), not an array
- **Segments vs Cohorts** use the same endpoint (`/segments`), differentiated only by the `type` field
- **Modal dialogs** in Umami block the Chrome extension entirely (screenshots, clicks, keyboard all fail) — always use the API via `javascript_tool` instead
- **Schema source**: `github.com/umami-software/umami` -> `src/lib/schema.ts` for Zod validation schemas
