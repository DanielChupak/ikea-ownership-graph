# Step 4 — Revoke-Permission Self-Service Action

**Goal:** A governed, approval-routed action that revokes a permission. For the
POC the effect is **simulated** — it flips the permission entity's
`status → revoked` in Port via an `UPSERT_ENTITY` invocation. No external IAM/
RBAC system is called.

**Action:** `ikea_revoke_permission`
**Type:** DAY-2 on `ikea_permission`
**Approval:** `requiredApproval: ANY` → routes to the permission's owning team
(inherited from the target API in Step 1), notified by email.

## Why this shape

- DAY-2 + `blueprintIdentifier: ikea_permission` → the action attaches to a
  specific permission entity (the one the agent proposes revoking).
- `UPSERT_ENTITY` invocation → Port itself writes `status: revoked` and
  `last_reviewed_at`, giving us a real, audited state change with zero backend.
- Swapping to real execution later = change `invocationMethod` to `WEBHOOK` /
  `GITHUB` pointing at IKEA's IAM automation. The action contract stays stable.

```
CallMcpTool server="user-port-eu" toolName="upsert_action" arguments=
```
```json
{
  "action": {
    "identifier": "ikea_revoke_permission",
    "title": "Revoke Permission",
    "icon": "Lock",
    "description": "Revoke this access grant. Routes to the target API owner for approval. (POC: simulated — flips status to revoked.)",
    "trigger": {
      "type": "self-service",
      "operation": "DAY-2",
      "blueprintIdentifier": "ikea_permission",
      "userInputs": {
        "properties": {
          "justification": {
            "title": "Reason for revocation",
            "type": "string",
            "format": "markdown"
          },
          "blast_radius_acknowledged": {
            "title": "I have reviewed the blast radius",
            "type": "boolean",
            "default": false
          }
        },
        "required": ["justification", "blast_radius_acknowledged"]
      }
    },
    "requiredApproval": { "type": "ANY" },
    "approvalNotification": { "type": "email" },
    "invocationMethod": {
      "type": "UPSERT_ENTITY",
      "blueprintIdentifier": "ikea_permission",
      "mapping": {
        "identifier": "{{ .entity.identifier }}",
        "properties": {
          "status": "revoked",
          "last_reviewed_at": "{{ .trigger.at }}",
          "justification": "{{ .inputs.justification }}"
        }
      }
    }
  }
}
```

> If your org's template engine rejects `{{ .trigger.at }}`, drop that line —
> `status: revoked` is the essential change. Confirm available context keys with
> `search_port_knowledge_sources` if a mapping token fails.

## Verify

```
CallMcpTool server="user-port-eu" toolName="run_action" arguments={
  "actionIdentifier": "ikea_revoke_permission",
  "entityIdentifier": "ikea-perm-fulfillment-to-orders",
  "properties": { "justification": "Dry-run test of approval gate.", "blast_radius_acknowledged": true },
  "executionMode": "manual"
}
```

Use `ikea-perm-fulfillment-to-orders` (a low-risk leaf) for the dry run so you
don't disturb the demo edge. `executionMode: manual` returns a link instead of
auto-executing.

**Checkpoint:** The action shows up as a DAY-2 action on any `ikea_permission`
entity, and triggering it creates a run that **waits for approval** rather than
executing immediately.

**Pause:** "Action is wired. Ready to build the dashboard (Step 5)?"

## Manual fallback

Self-service → **+ Action** → Edit JSON → paste the `action` object above.
Confirm the Approval tab shows "Require approval = Any approver".
