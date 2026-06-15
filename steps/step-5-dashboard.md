# Step 5 — Command-Center Dashboard

**Goal:** One page where owners and security see the graph, the risky
permissions, the open vulns, and can launch a revoke — plus the agent inline.

**Page:** `ikea_ownership_graph`

> Widget payloads vary by Port version. If any widget is rejected, run
> `load_widget_schema` for that `type` and adjust. The layout rule is strict:
> **each row's column `size` values must sum to exactly 12**, and the number of
> layout columns must match the number of widgets.

## Widgets

1. `hdr` — markdown intro
2. `agent` — `ai-agent` (the Step 6 agent; create the page after the agent, or
   create now and the widget fills in once the agent exists)
3. `perms` — `table-entities-explorer` over `ikea_permission`
4. `open_vulns` — `entities-number-chart` counting open `ikea_vuln_finding`
5. `scope_mix` — `entities-pie-chart` of permissions by `scope`
6. `revoke` — `action-card-widget` exposing `ikea_revoke_permission`

```
CallMcpTool server="user-port-eu" toolName="upsert_dashboard_page" arguments=
```
```json
{
  "identifier": "ikea_ownership_graph",
  "title": "IKEA Ownership & Access Graph",
  "icon": "Lock",
  "layout": [
    { "height": 400, "columns": [ { "id": "hdr", "size": 12 } ] },
    { "height": 600, "columns": [ { "id": "agent", "size": 7 }, { "id": "open_vulns", "size": 5 } ] },
    { "height": 500, "columns": [ { "id": "perms", "size": 8 }, { "id": "scope_mix", "size": 4 } ] },
    { "height": 400, "columns": [ { "id": "revoke", "size": 12 } ] }
  ],
  "widgets": [
    {
      "id": "hdr",
      "type": "markdown",
      "title": "Ownership-Aware Access Graph",
      "markdown": "## Ask, don't dig\nAsk the agent who owns a service/API, what it can access, what vulns are open, and **what would change if you revoked a permission**. Review the blast radius, then roll out the revoke with owner approval — all from this page."
    },
    {
      "id": "agent",
      "type": "ai-agent",
      "title": "Blast-Radius Assistant",
      "agentIdentifier": "ikea_blast_radius_agent"
    },
    {
      "id": "open_vulns",
      "type": "entities-number-chart",
      "title": "Open Vuln Findings",
      "icon": "Alert",
      "calculationBy": "entities",
      "blueprint": "ikea_vuln_finding",
      "func": "count",
      "filters": [ { "combinator": "and", "rules": [ { "property": "status", "operator": "=", "value": "open" } ] } ]
    },
    {
      "id": "perms",
      "type": "table-entities-explorer",
      "title": "Permissions (access edges)",
      "blueprint": "ikea_permission"
    },
    {
      "id": "scope_mix",
      "type": "entities-pie-chart",
      "title": "Permissions by Scope",
      "blueprint": "ikea_permission",
      "property": "property#scope"
    },
    {
      "id": "revoke",
      "type": "action-card-widget",
      "title": "Roll out a change",
      "actions": [ { "actionIdentifier": "ikea_revoke_permission" } ]
    }
  ]
}
```

## Verify

Open the page from the Port sidebar. Confirm the permission table is populated,
the open-vuln count is non-zero, and the pie chart shows the read/write/admin mix.

**Checkpoint:** Dashboard renders with live entity data. (The agent widget shows
an empty state until Step 6 creates the agent — that's expected.)

**Pause:** "Dashboard is up. Ready for the AI agent (Step 6)?"

## Notes
- If `entities-number-chart` rejects `calculationBy`/`func`, call
  `load_widget_schema type="entities-number-chart"` and match the exact keys.
- `property#scope` is the common Port convention for charting an enum property;
  if rejected, check the pie-chart schema via `load_widget_schema`.
