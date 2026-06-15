# VALIDATION — IKEA Ownership-Aware Access Graph

End-to-end checklist. Run top to bottom; each step must pass before the next.

## Pre-flight
- [ ] MCP server `user-port-eu` reachable (`list_blueprints` returns data).
- [ ] `integrations` empty → confirms dummy-data path (or customer accepted
      dummy data anyway).
- [ ] Existing `service`, `_team`, `_user` blueprints present (we reuse them).

## Step 1 — Catalog
- [ ] `ikea_api` created (Direct ownership; `exposed_by → service`).
- [ ] `ikea_permission` created (`consumer → service`, `target_api → ikea_api`;
      Inherited ownership via `target_api`).
- [ ] `ikea_vuln_finding` created (`affected_service`, `affected_api`).
- [ ] Created in order: `ikea_api` before the other two.
- [ ] `list_blueprints identifiers=[...]` shows all relations resolving.

## Step 2 — Seed graph
- [ ] Owner teams exist (reused or `ikea-` prefixed).
- [ ] 5 `ikea-` services created.
- [ ] 3 `ikea_api` entities created, each `exposed_by` its service.
- [ ] 4 `ikea_permission` edges created (the chain in step-2 file).
- [ ] ≥3 `ikea_vuln_finding` entities created, ≥1 critical + open.
- [ ] Opening `ikea-checkout-service` shows owner + reachable API + findings.

## Step 3 — Scorecard
- [ ] `ikea_access_hygiene` created on `ikea_permission`.
- [ ] No rule assigned to `Basic` level (else 409).
- [ ] Seeded permissions display Bronze/Silver/Gold levels.

## Step 4 — Action
- [ ] `ikea_revoke_permission` created (DAY-2 on `ikea_permission`).
- [ ] `requiredApproval: ANY` present.
- [ ] Invocation is `UPSERT_ENTITY` writing `status: revoked` (simulated).
- [ ] Manual dry-run on a leaf permission shows the approval gate (does not
      auto-execute).

## Step 5 — Dashboard
- [ ] Page `ikea_ownership_graph` created.
- [ ] Each layout row's column sizes sum to 12; widget count matches layout.
- [ ] Permission table populated; open-vuln count > 0; scope pie renders.
- [ ] Revoke action card present.

## Step 6 — Agent
- [ ] `ikea_blast_radius_agent` entity created on `_ai_agent`.
- [ ] Agent answers ownership / access / vuln questions over the graph.
- [ ] (Optional) `ikea_on_permission_revoked` automation created with a
      placeholder (`.invalid`) webhook.

## Step 7 — Demo
- [ ] Agent computes blast radius for revoking checkout→payments (multi-hop).
- [ ] Agent names affected services + owners + open vulns.
- [ ] Agent offers + runs `ikea_revoke_permission`; run waits for approval.
- [ ] On approval, `ikea-perm-checkout-to-payments` → `status: revoked`.
- [ ] Predicted affected set == actual graph traversal.

## Build-quality gates (from the builder skill)
- [ ] Skill name matches `{customer}-{story}` → `ikea-ownership-graph`. ✓
- [ ] All MCP tool names match `user-port-eu` schemas.
- [ ] Blueprint + entity creation order documented.
- [ ] Demo narrative has concrete trigger + expected outcome. ✓
- [ ] No destructive operations without explicit consent. ✓
- [ ] README includes Cursor MCP setup. ✓
- [ ] SKILL.md includes `Behavior — CRITICAL`. ✓
- [ ] Each step has checkpoint + pause. ✓
- [ ] No instructions that batch-build all steps. ✓
- [ ] SKILL.md under 500 lines. ✓

---

## Post-POC — Phase 2 (deferred, not built here)

Spelled out in the user story as "follow post-POC":
- [ ] **Agents as first-class assets** in the graph (with their own permissions).
- [ ] **Data products** as nodes + ownership + access.
- [ ] **Infrastructure** (cloud resources, K8s) unified into the graph.
- [ ] **Access logs** as a signal (who actually used a permission).
- [ ] **Policy-tightening simulations** beyond revoke (scope reduction,
      conditional access, time-boxing).
- [ ] **Real execution** — swap the simulated `UPSERT_ENTITY` invocation for a
      `WEBHOOK`/`GITHUB` call into IKEA's IAM automation.
- [ ] **Real integrations** — replace dummy data (GitHub ownership, IAM/RBAC
      source, Snyk/Wiz vuln findings) via Port integrations.
