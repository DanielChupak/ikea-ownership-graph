---
name: ikea-ownership-graph
description: >-
  Guided Port onboarding for IKEA security engineers: build a single,
  ownership-aware graph of services and APIs unified with RBAC permissions and
  open vulnerability findings, then ask the Port agent in natural language who
  owns what, what it can access, what vulns are open, and — critically — what
  would change if a specific permission were revoked (blast radius), with a
  one-click, approval-routed self-service action to roll out the change.
  Use when starting the IKEA ownership-graph POC onboarding in Port.
---

# IKEA — Ownership-Aware Access Graph (POC)

An interactive, step-by-step workshop that stands up the IKEA POC in Port:
a unified graph where **services + APIs** carry **ownership + RBAC + open vuln
findings**, queryable in natural language, with **"revoke permission"** as the
first what-if scenario — computing blast radius and executing the change as an
approval-routed self-service action.

> **Persona:** Security engineer at IKEA
> **Region:** EU (`user-port-eu` MCP server)
> **Data:** Dummy, `ikea-` prefixed (no integrations connected yet)
> **Execution:** Revoke is *simulated* — it flips the permission's state in the
> graph via Port (`UPSERT_ENTITY`); no external system is called.

---

## Behavior — CRITICAL

These rules are non-negotiable. Read them before doing anything else.

1. **This is a workshop, not a batch installer.** Guide the user through **one
   step at a time**. Explain → build together → checkpoint → pause.
2. **On activation**, show the welcome + journey table + step count, then **wait
   for the user to say "let's go"** (or similar) before starting Step 0.
3. **Run MCP tools for the CURRENT step only.** Never batch Steps 1–N in one
   turn. After each step, **stop and wait** for the user to confirm before
   moving on.
4. **"create it for me" / "do it for me"** runs the MCP calls for the **current
   step only**, then pauses at the checkpoint.
5. **Read tool schemas before calling.** Tool descriptors live under the MCP
   filesystem; the per-step files include the exact payloads, but always
   confirm the schema if a call fails.
6. **Adapt, don't overwrite.** This org already has `service`, `_team`, `_user`,
   `githubTeam`, etc. **Reuse** them. New POC blueprints are prefixed `ikea_`
   and dummy entities are prefixed `ikea-`. **Never delete** existing resources.
7. **Demo data is built last.** Don't pre-create demo entities until the demo
   step, except the baseline graph seeded in Step 2.
8. **Manual fallback always available.** Each step file contains copy-paste JSON
   for the Port UI if the user prefers clicking over MCP.

---

## Activation Script

When the user starts this skill, respond with the welcome below (adapt tone,
keep it tight), then **stop and wait**.

> **Welcome — IKEA Ownership-Aware Access Graph POC**
>
> We'll build a single graph in Port where every **service** and **API** carries
> its **owner**, its **permissions** (what it can access), and its **open vuln
> findings** — then make it answerable in plain English, including the big one:
> *"What breaks if I revoke this permission?"*
>
> Here's the journey:
>
> | Step | What we build | Port artifact |
> |------|---------------|---------------|
> | 0 | Orientation + connectivity check | — |
> | 1 | Catalog foundation | Blueprints: `ikea_api`, `ikea_permission`, `ikea_vuln_finding` |
> | 2 | Seed the ownership graph | Entities (services, APIs, permissions, findings, owners) |
> | 3 | Access-hygiene scorecard | Scorecard on `ikea_permission` |
> | 4 | Revoke-permission action | Approval-routed self-service action (simulated) |
> | 5 | Command-center dashboard | Dashboard page + widgets |
> | 6 | Natural-language agent | AI agent + (optional) automation |
> | 7 | Live what-if demo | End-to-end blast-radius + revoke |
>
> **8 steps.** We'll do them one at a time — I'll explain, we build together,
> then I pause for your OK before moving on.
>
> Ready for **Step 0**?

---

## Step Orchestration (inline)

For each step: state the **Problem**, **What we build**, run the step
(**Do together** — pull commands from the step file), hit the **Checkpoint**,
then **Pause**. Full MCP payloads + manual JSON live in `steps/`.

### Step 0 — Orientation & connectivity
- **Problem:** Before building, confirm we're talking to the right Port org.
- **Do together:** Run `list_blueprints` (summary) on `user-port-eu`. Confirm
  `service`, `_team`, `_user` exist and `integrations` is empty (dummy-data path).
- **Checkpoint:** User sees the blueprint list returned.
- **Pause:** "Connected. Ready for Step 1 — the catalog foundation?"
- Details: [steps/step-1-catalog.md](steps/step-1-catalog.md) (Step 0 is the preamble there).

### Step 1 — Catalog foundation
- **Problem:** The graph needs nodes for APIs, the revocable permission edge, and
  vuln findings. Services and owners already exist — we reuse them.
- **What we build:** 3 blueprints — `ikea_api`, `ikea_permission`,
  `ikea_vuln_finding` — with relations that wire the graph together.
- **Do together:** `upsert_blueprint` ×3 in dependency order (`ikea_api` first,
  since `ikea_permission` and `ikea_vuln_finding` relate to it).
- **Checkpoint:** All three appear in `list_blueprints`; relations resolve.
- **Pause:** "Catalog is live. Ready to seed the graph (Step 2)?"
- Details: [steps/step-1-catalog.md](steps/step-1-catalog.md)

### Step 2 — Seed the ownership graph
- **Problem:** An empty graph proves nothing. We need a small but realistic
  slice: a few owned services, the APIs they expose, the permissions between
  them, and some open vulns.
- **What we build:** `ikea-` prefixed entities forming a multi-hop permission
  chain (so blast radius is non-trivial).
- **Do together:** `upsert_entity` for teams/owners → services → APIs →
  permissions → vuln findings (relation order matters).
- **Checkpoint:** Open a service entity; see owner, exposed APIs, inbound/outbound
  permissions, and findings in one view.
- **Pause:** "Graph is populated. Ready for the hygiene scorecard (Step 3)?"
- Details: [steps/step-2-seed-graph.md](steps/step-2-seed-graph.md)

### Step 3 — Access-hygiene scorecard
- **Problem:** Security wants to know which permissions are risky *before* a
  change is even proposed.
- **What we build:** `ikea_access_hygiene` scorecard on `ikea_permission`
  (least-privilege, justification present, not stale).
- **Do together:** `upsert_scorecard` on `ikea_permission`.
- **Checkpoint:** Permissions show Bronze/Silver/Gold levels.
- **Pause:** "Hygiene scoring is on. Ready to build the revoke action (Step 4)?"
- Details: [steps/step-3-scorecard.md](steps/step-3-scorecard.md)

### Step 4 — Revoke-permission self-service action
- **Problem:** Knowing the blast radius is useless if rolling out the change is
  still a manual ticket. We need a governed, approval-routed action.
- **What we build:** `ikea_revoke_permission` — a DAY-2 action on
  `ikea_permission`, `requiredApproval: ANY`, that flips `status → revoked`
  (simulated via `UPSERT_ENTITY`).
- **Do together:** `upsert_action`.
- **Checkpoint:** Action appears on a permission entity; dry-run shows the
  approval gate.
- **Pause:** "Action is wired. Ready to build the dashboard (Step 5)?"
- Details: [steps/step-4-revoke-action.md](steps/step-4-revoke-action.md)

### Step 5 — Command-center dashboard
- **Problem:** Owners and security need one place to see the graph, the risky
  permissions, and the open vulns.
- **What we build:** "IKEA Ownership & Access Graph" dashboard with the agent,
  permission table, vuln metrics, and the revoke action card.
- **Do together:** `upsert_dashboard_page` (use `load_widget_schema` if a widget
  payload is unclear).
- **Checkpoint:** Dashboard renders with live entity data.
- **Pause:** "Dashboard is up. Ready for the AI agent (Step 6)?"
- Details: [steps/step-5-dashboard.md](steps/step-5-dashboard.md)

### Step 6 — Natural-language blast-radius agent
- **Problem:** Security engineers shouldn't have to learn the data model — they
  should just ask.
- **What we build:** `ikea_blast_radius_agent` (entity of `_ai_agent`) that
  traverses the permission graph, computes blast radius, surfaces owners +
  vulns, and offers to run `ikea_revoke_permission`. Optional automation to
  notify owners on revoke.
- **Do together:** `upsert_entity` on `_ai_agent`; optional `upsert_action`
  (automation trigger).
- **Checkpoint:** Agent answers "who owns X / what can X access / open vulns".
- **Pause:** "Agent is live. Ready for the demo (Step 7)?"
- Details: [steps/step-6-agent.md](steps/step-6-agent.md)

### Step 7 — Live what-if demo
- **Problem:** Prove the whole loop in one breath.
- **Do together:** Ask the agent: *"What would change if I revoked
  `ikea-checkout-service`'s write access to the payments API?"* → it lists
  affected downstream services + owners + open vulns and the blast radius →
  offers the revoke action → approval routes to the API owner → on approval the
  permission flips to `revoked` and the dashboard updates.
- **Checkpoint:** The graph reflects the revoked edge; affected set matched the
  agent's prediction.
- **Done:** POC value delivered — what took days of manual analysis is now
  minutes with full impact visibility.
- Details: [steps/step-7-demo.md](steps/step-7-demo.md)

---

## Data Model Summary

```
                 owns                       exposes
  _team / _user ──────▶ service ◀───────────────────── ikea_api
        ▲                  ▲   ▲                            ▲
        │ owner            │   │ consumer        target_api │
        │                  │   └──────────── ikea_permission ┘
        │                  │                    (scope, status)
        │       affected_service                      
        └───────────── ikea_vuln_finding ──────── affected_api
```

- **service** (existing, reused): Direct ownership, `tier`, `criticality`.
- **ikea_api**: `exposed_by → service`; Direct ownership; `classification`,
  `lifecycle`.
- **ikea_permission**: `consumer → service`, `target_api → ikea_api`; `scope`
  (read/write/admin), `status` (active/pending_revocation/revoked),
  `justification`, `risk_level`, `granted_at`. **This is the revocable edge.**
- **ikea_vuln_finding**: `affected_service → service`, `affected_api → ikea_api`;
  `cve`, `severity`, `cvss_score`, `status`.

**Blast-radius logic (how the agent traverses):** revoking permission `P`
(consumer `A` → `target_api` exposed by provider `B`) directly cuts `A`'s access
to `B`. The blast radius is found by walking the permission edges: find every
API `exposed_by A`, then every permission whose `target_api` is one of those —
those consumers are the 1-hop blast radius; recurse for multi-hop. Owners and
open vuln findings of every affected service/API are surfaced alongside.

---

## MCP Tool Cheat-Sheet (server: `user-port-eu`)

| Operation | Tool |
|-----------|------|
| Inspect | `list_blueprints`, `list_entities`, `list_integrations` |
| Catalog | `upsert_blueprint`, `upsert_entity` |
| Scorecard | `upsert_scorecard` |
| Action | `upsert_action`, `run_action`, `track_action_run` |
| Dashboard | `upsert_dashboard_page`, `upsert_widget`, `load_widget_schema` |
| Agent | `upsert_entity` (blueprint `_ai_agent`), `load_skill` |

Always read the tool descriptor before a first-time call. Full payloads are in
the `steps/` files.

---

## Safety Rules (POC)

1. **Never delete** existing IKEA resources.
2. All new blueprints are `ikea_`-prefixed; all dummy entities are
   `ikea-`-prefixed.
3. Revoke is **simulated** (entity state change only) for the POC.
4. Inspect before create (`list_blueprints` first).
5. One step's MCP calls per turn — never batch.

---

## Out of Scope (post-POC)

Agents-as-assets, data products, infrastructure, access logs, and
policy-tightening simulations are explicitly **deferred**. This POC covers
services + APIs, ownership + RBAC + open vuln findings, and *revoke permission*
as the single what-if. See [VALIDATION.md](VALIDATION.md) for the Phase-2 list.
