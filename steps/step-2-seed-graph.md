# Step 2 — Seed the Ownership Graph

**Goal:** Populate a small but realistic slice so blast radius is non-trivial.
We build a multi-hop permission chain across owned services and APIs, plus a few
open vulns.

> All entities are `ikea-` prefixed so they never collide with real data.

## The scenario we're seeding

```
ikea-checkout-service ──write──▶ payments API (owned by Payments team)
ikea-orders-service   ──read───▶ checkout API (owned by Checkout team)
ikea-fulfillment-svc  ──read───▶ orders API   (owned by Orders team)
ikea-analytics-svc    ──read───▶ orders API
```

So revoking **checkout → payments (write)** degrades `ikea-checkout-service`,
which `ikea-orders-service` depends on, which `ikea-fulfillment-svc` and
`ikea-analytics-svc` depend on → a 3-hop blast radius. Perfect for the demo.

## Creation order (relations must resolve)

1. Teams (owners) → reuse `_team` if present, else create `ikea-`-prefixed teams.
2. Services (`service` blueprint, `ikea-` prefixed).
3. APIs (`ikea_api`, related to their exposing service).
4. Permissions (`ikea_permission`, related to consumer + target_api).
5. Vuln findings (`ikea_vuln_finding`).

> Tip: pass `createMissingRelatedEntities: true` on `upsert_entity` to reduce
> ordering pain, but the order below is the safe path.

---

## 2a. Teams (owners)

If reusing existing teams, skip. Otherwise create lightweight owners:

```
CallMcpTool server="user-port-eu" toolName="upsert_entity" arguments={
  "blueprintIdentifier": "_team",
  "entity": { "identifier": "ikea-payments-team", "title": "IKEA Payments Team", "properties": {} }
}
```
Repeat for `ikea-checkout-team`, `ikea-orders-team`, `ikea-platform-team`.

> If `_team` requires fields your org enforces, run
> `list_blueprints identifiers=["_team"]` and fill required props.

---

## 2b. Services (reuse `service` blueprint)

```
CallMcpTool server="user-port-eu" toolName="upsert_entity" arguments={
  "blueprintIdentifier": "service",
  "entity": {
    "identifier": "ikea-checkout-service",
    "title": "IKEA Checkout Service",
    "team": ["ikea-checkout-team"],
    "properties": { "criticality": "high", "tier": "tier-1" }
  }
}
```

Create: `ikea-checkout-service`, `ikea-payments-service`, `ikea-orders-service`,
`ikea-fulfillment-service`, `ikea-analytics-service`.

> Confirm `criticality`/`tier` accepted values with
> `list_blueprints identifiers=["service"]` first; adjust if the enums differ.
> If a property errors, omit it — only identifier/title/team are essential here.

---

## 2c. APIs (`ikea_api`)

```
CallMcpTool server="user-port-eu" toolName="upsert_entity" arguments={
  "blueprintIdentifier": "ikea_api",
  "entity": {
    "identifier": "ikea-payments-api",
    "title": "Payments API",
    "team": ["ikea-payments-team"],
    "properties": { "domain": "payments", "classification": "restricted", "lifecycle": "production", "base_url": "https://api.ikea.example/payments" },
    "relations": { "exposed_by": "ikea-payments-service" }
  }
}
```

Create:
| API | exposed_by | classification |
|-----|-----------|----------------|
| `ikea-payments-api` | `ikea-payments-service` | restricted |
| `ikea-checkout-api` | `ikea-checkout-service` | confidential |
| `ikea-orders-api` | `ikea-orders-service` | internal |

---

## 2d. Permissions (`ikea_permission`) — the edges

```
CallMcpTool server="user-port-eu" toolName="upsert_entity" arguments={
  "blueprintIdentifier": "ikea_permission",
  "entity": {
    "identifier": "ikea-perm-checkout-to-payments",
    "title": "checkout-service → payments-api (write)",
    "properties": { "scope": "write", "status": "active", "risk_level": "high", "justification": "Submit and capture payments at checkout.", "granted_at": "2025-09-01T09:00:00Z" },
    "relations": { "consumer": "ikea-checkout-service", "target_api": "ikea-payments-api" }
  }
}
```

Create all edges:
| Permission id | consumer | target_api | scope | risk |
|---------------|----------|------------|-------|------|
| `ikea-perm-checkout-to-payments` | checkout-service | payments-api | write | high |
| `ikea-perm-orders-to-checkout` | orders-service | checkout-api | read | medium |
| `ikea-perm-fulfillment-to-orders` | fulfillment-service | orders-api | read | low |
| `ikea-perm-analytics-to-orders` | analytics-service | orders-api | read | low |

> Note the chain: payments-api is exposed_by payments-service, but checkout-api
> is exposed_by checkout-service and orders-api by orders-service — that's what
> creates the multi-hop dependency for blast radius.

---

## 2e. Vuln findings (`ikea_vuln_finding`)

```
CallMcpTool server="user-port-eu" toolName="upsert_entity" arguments={
  "blueprintIdentifier": "ikea_vuln_finding",
  "entity": {
    "identifier": "ikea-vuln-payments-001",
    "title": "CVE-2025-12345 in payments-service",
    "properties": { "cve": "CVE-2025-12345", "severity": "critical", "cvss_score": 9.1, "status": "open", "package": "openssl@3.0.1", "description": "RCE in TLS handshake path." },
    "relations": { "affected_service": "ikea-payments-service", "affected_api": "ikea-payments-api" }
  }
}
```

Add 2–3 more (mixed severities) across `ikea-checkout-service` and
`ikea-orders-service` so the dashboard metrics are non-empty.

---

## Verify

```
CallMcpTool server="user-port-eu" toolName="list_entities" arguments={ "blueprintIdentifier": "ikea_permission" }
```

**Checkpoint:** Open `ikea-checkout-service` in the UI → you should see its team
(owner), the `ikea-payments-api` it can reach (via the permission), and any open
findings. The graph is now traversable.

**Pause:** "Graph is populated. Ready for the hygiene scorecard (Step 3)?"
