# Step 6 — Natural-Language Blast-Radius Agent

**Goal:** A Port AI agent that lets a security engineer ask, in plain English,
who owns a service/API, what it can access, what vulns are open, and **what would
change if a specific permission were revoked** — then offers to run the revoke
action with approval routing.

**Blueprint:** `_ai_agent` (system blueprint, already in the org)
**Entity:** `ikea_blast_radius_agent`

> Agents are created as **entities of `_ai_agent`**. Confirm the exact property
> keys first: `list_blueprints identifiers=["_ai_agent"]`. The payload below uses
> the standard keys (`prompt`, `tools`, `execution_mode`, `conversation_starters`).
> Adjust names if your org's `_ai_agent` differs.

```
CallMcpTool server="user-port-eu" toolName="upsert_entity" arguments=
```
```json
{
  "blueprintIdentifier": "_ai_agent",
  "entity": {
    "identifier": "ikea_blast_radius_agent",
    "title": "Blast-Radius Assistant",
    "properties": {
      "status": "active",
      "execution_mode": "Approval Required",
      "description": "Ownership-aware access-graph assistant for IKEA security engineers.",
      "prompt": "You are IKEA's ownership-aware access-graph assistant. You operate over Port blueprints: `service` (with owning teams), `ikea_api` (exposed_by a service), `ikea_permission` (consumer service -> target_api, with scope and status), and `ikea_vuln_finding` (affected_service / affected_api).\n\nYou can answer four classes of question:\n1. OWNERSHIP — 'who owns service/API X?' Read the owning team(s).\n2. ACCESS — 'what can X access / who can access X?' Traverse ikea_permission edges (consumer and target_api).\n3. RISK — 'what open vulns are against X?' Read ikea_vuln_finding where status=open.\n4. WHAT-IF (BLAST RADIUS) — 'what changes if I revoke permission P?'\n\nBLAST-RADIUS ALGORITHM (do this explicitly and show your work):\n- Identify permission P: consumer A -> target_api B_api (exposed_by provider B).\n- Direct impact: A loses `scope` access to B_api.\n- Find APIs exposed_by A. For each, find ikea_permission edges whose target_api is one of those APIs. Their consumers are the 1-hop blast radius (they depend on A).\n- Recurse on those consumers to find multi-hop impact. Stop at a sensible depth and list the full affected set.\n- For every affected service/API, surface: owning team(s) and any open ikea_vuln_finding.\n- Present a concise impact summary: directly cut access, downstream services at risk, owners to notify, and open vulns in the affected set.\n\nThen OFFER to roll out the change by running the `ikea_revoke_permission` self-service action on permission P. Never execute without explicit user confirmation. Remind the user the action routes to the target API owner for approval. For the POC, revocation is simulated (flips status to revoked in Port).\n\nAlways cite the entity identifiers you traversed so the engineer can verify.",
      "conversation_starters": [
        "Who owns the payments API and what can reach it?",
        "What would change if I revoked ikea-checkout-service's write access to the payments API?",
        "Show open critical vulns on services that can access the orders API."
      ]
    }
  }
}
```

> `execution_mode` values vary by org (e.g. `Approval Required` vs `Automatic`).
> If rejected, run the blueprint inspect command above and use a listed enum
> value. Same for `status`.

## Verify

Open the agent in Port (AI → Agents) or use the `ai-agent` widget on the Step 5
dashboard. Ask: *"Who owns the payments API?"* — it should answer from the graph.

**Checkpoint:** Agent answers ownership / access / vuln questions over the seeded
graph and references entity identifiers.

**Pause:** "Agent is live. Ready for the demo (Step 7)?"

---

## Optional — notify owners on revocation (automation)

Closes the loop: when a permission flips to `revoked`, notify. POC-safe version
just records it; point the webhook at a real Slack/email later.

```
CallMcpTool server="user-port-eu" toolName="upsert_action" arguments=
```
```json
{
  "action": {
    "identifier": "ikea_on_permission_revoked",
    "title": "On Permission Revoked",
    "icon": "Bell",
    "trigger": {
      "type": "automation",
      "event": { "type": "ENTITY_UPDATED", "blueprintIdentifier": "ikea_permission" },
      "condition": {
        "type": "JQ",
        "expressions": [ ".diff.after.properties.status == \"revoked\"", ".diff.before.properties.status != \"revoked\"" ],
        "combinator": "and"
      }
    },
    "invocationMethod": {
      "type": "WEBHOOK",
      "url": "https://example.ikea.invalid/notify",
      "body": { "message": "Permission {{ .event.diff.after.identifier }} was revoked", "permission": "{{ .event.diff.after.identifier }}" }
    },
    "publish": true
  }
}
```

> The webhook URL is a placeholder (`.invalid`) so it won't fire externally
> during the POC. Replace with IKEA's real endpoint when going beyond POC.
