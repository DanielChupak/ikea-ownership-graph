# Step 7 — Live What-If Demo

**Goal:** Prove the whole loop in one breath — natural-language question →
blast-radius computation → owner-routed approval → graph updates.

This is the payoff: what used to take **~N days of manual analysis with residual
risk** is now **~M minutes with full impact visibility**.

> Don't pre-stage anything new — Steps 1–6 already built the graph, scorecard,
> action, dashboard, and agent. Just run the scenario.

## The scenario

Recall the seeded chain:

```
ikea-checkout-service ──write──▶ payments-api
ikea-orders-service   ──read───▶ checkout-api   (exposed_by checkout-service)
ikea-fulfillment-svc  ──read───▶ orders-api     (exposed_by orders-service)
ikea-analytics-svc    ──read───▶ orders-api
```

Revoking **checkout → payments (write)** degrades `ikea-checkout-service`, which
`ikea-orders-service` depends on, which fulfillment + analytics depend on.

## Run it

1. **Open the dashboard** (`IKEA Ownership & Access Graph`) → use the
   **Blast-Radius Assistant** widget (or the agent directly).

2. **Ask:**
   > "What would change if I revoked `ikea-checkout-service`'s write access to
   > the payments API?"

3. **Expected agent response:**
   - **Permission:** `ikea-perm-checkout-to-payments` (write, status active).
   - **Direct impact:** `ikea-checkout-service` loses write to `ikea-payments-api`
     (owned by IKEA Payments Team).
   - **Blast radius (downstream):** `ikea-orders-service` (reads checkout-api),
     then `ikea-fulfillment-service` and `ikea-analytics-service` (read
     orders-api).
   - **Owners to notify:** Checkout, Orders, Payments teams.
   - **Open vulns in the affected set:** e.g. `CVE-2025-12345` (critical) on
     `ikea-payments-service`.
   - **Offer:** "Shall I roll this out via `ikea_revoke_permission`? It routes to
     the Payments team for approval."

4. **Confirm.** The agent runs `ikea_revoke_permission` on
   `ikea-perm-checkout-to-payments`. The run **waits for approval** (requiredApproval: ANY).

5. **Approve** (as the API owner) in Port → the `UPSERT_ENTITY` invocation flips
   the permission `status → revoked`.

6. **Watch the graph update:** the permission table on the dashboard now shows
   the edge as `revoked`; the optional automation (Step 6) fires the
   (placeholder) notification.

## Verify

```
CallMcpTool server="user-port-eu" toolName="list_entities" arguments={ "blueprintIdentifier": "ikea_permission" }
```

**Checkpoint:** `ikea-perm-checkout-to-payments` shows `status: revoked`, and the
set of affected services matches what the agent predicted in step 3 above.

## Talk track (for the SE running the POC)

> "We asked a plain-English question. Port traversed the ownership + RBAC graph,
> computed the blast radius across three hops, told us exactly which owners to
> involve and which open vulns sit in the affected set, and let us execute the
> change with approval routing — without leaving the page. That's days of
> manual cross-team analysis collapsed into minutes, with the impact visible up
> front."

## Reset (optional)

To re-run the demo, set the permission back to active:

```
CallMcpTool server="user-port-eu" toolName="upsert_entity" arguments={
  "blueprintIdentifier": "ikea_permission",
  "entity": { "identifier": "ikea-perm-checkout-to-payments", "properties": { "status": "active" } }
}
```
