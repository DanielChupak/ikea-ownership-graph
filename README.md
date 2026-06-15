# IKEA Ownership-Aware Access Graph — Setup

This is a **guided, run-it-yourself onboarding skill** for Port. It walks a
security engineer through building a single graph where services and APIs carry
their ownership, RBAC permissions, and open vulnerability findings — then makes
it answerable in natural language, with "revoke permission" as the first
what-if scenario (blast radius + approval-routed self-service action).

## Prerequisites

- **Cursor** (or another agent host that supports MCP + skills).
- A **Port** account on the **EU** region with permission to create blueprints,
  entities, scorecards, actions, dashboards, and AI agents.
- Port **MCP server** connected as `user-port-eu`.

## 1. Connect the Port MCP server

Add the Port EU MCP server to your Cursor MCP config (Settings → MCP, or
`~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "user-port-eu": {
      "url": "https://mcp.port.io/v1",
      "headers": { "Authorization": "Bearer <YOUR_PORT_API_TOKEN>" }
    }
  }
}
```

> Generate the token in Port (Settings → Credentials / API). The server id
> **must** be `user-port-eu` — every step in this skill targets that id.
> If your Port org is on the US region instead, this skill won't match; ask
> your Port SE for the US variant.

Verify the connection works by asking the agent to run `list_blueprints` against
`user-port-eu`.

## 2. Install the skill

Copy this whole folder to your Cursor skills directory:

```
~/.cursor/skills/ikea-ownership-graph/
```

(That's where it already lives if your SE handed it to you in place.)

## 3. Start the workshop

Open Cursor and say:

> "Start the IKEA ownership-graph onboarding."

The agent will greet you, show the 8-step journey, and wait for your go-ahead.
It builds **one step at a time** and pauses for your confirmation after each —
nothing is created in bulk.

## What gets built

| # | Step | Artifact |
|---|------|----------|
| 0 | Orientation | Connectivity check |
| 1 | Catalog | `ikea_api`, `ikea_permission`, `ikea_vuln_finding` blueprints |
| 2 | Seed graph | `ikea-` prefixed services, APIs, permissions, findings |
| 3 | Scorecard | `ikea_access_hygiene` on permissions |
| 4 | Action | `ikea_revoke_permission` (approval-routed, simulated) |
| 5 | Dashboard | "IKEA Ownership & Access Graph" page |
| 6 | Agent | `ikea_blast_radius_agent` + optional automation |
| 7 | Demo | End-to-end blast-radius what-if |

## Safety

- This skill **only creates** resources; it never deletes your existing data.
- New blueprints are prefixed `ikea_`; all sample entities are prefixed `ikea-`.
- The revoke action is **simulated** (it flips a status field in Port) — no
  external IAM/RBAC system is touched during the POC.
- Prefer the manual path? Every step file includes copy-paste JSON for the Port
  UI.

## Scope

**In:** services + APIs, ownership + RBAC + open vuln findings, "revoke
permission" what-if.
**Deferred (post-POC):** agents-as-assets, data products, infrastructure,
access logs, policy-tightening simulations.

## Troubleshooting

- *MCP calls fail with auth errors* → regenerate the Port token; re-check the
  `Authorization` header.
- *A property/enum is rejected* → run `list_blueprints` with the blueprint
  identifier to see the exact accepted values, then adjust.
- *A widget is rejected* → run `load_widget_schema` for that widget type and
  match keys.
- *A mapping token (e.g. `{{ .trigger.at }}`) errors* → remove it; the core
  status change still works.
