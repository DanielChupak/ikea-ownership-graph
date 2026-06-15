# Step 1 — Catalog Foundation (incl. Step 0 preamble)

**Goal:** Create the three POC blueprints that turn the existing service/owner
data into an ownership-aware *access graph*: `ikea_api`, `ikea_permission`,
`ikea_vuln_finding`.

> Reuse, don't recreate: `service`, `_team`, `_user`, `githubTeam` already exist
> in this org. We relate to them; we never modify or delete them.

---

## Step 0 — Connectivity check (run first)

```
CallMcpTool server="user-port-eu" toolName="list_blueprints" arguments={}
CallMcpTool server="user-port-eu" toolName="list_integrations" arguments={}
```

**Expect:** `service`, `_team`, `_user` present; `integrations: []` (confirms the
dummy-data path). Pause and confirm before continuing.

---

## Dependency order

1. `ikea_api` — no dependency on the other two.
2. `ikea_permission` — relates to `service` (consumer) and `ikea_api` (target).
3. `ikea_vuln_finding` — relates to `service` and `ikea_api`.

Create `ikea_api` **first**.

---

## 1a. `ikea_api`

```
CallMcpTool server="user-port-eu" toolName="upsert_blueprint" arguments=
```
```json
{
  "blueprint": {
    "identifier": "ikea_api",
    "title": "IKEA API",
    "icon": "Swagger",
    "schema": {
      "properties": {
        "domain": { "title": "Domain", "type": "string" },
        "base_url": { "title": "Base URL", "type": "string", "format": "url" },
        "classification": {
          "title": "Data Classification", "type": "string",
          "enum": ["public", "internal", "confidential", "restricted"],
          "enumColors": { "public": "green", "internal": "blue", "confidential": "orange", "restricted": "red" }
        },
        "lifecycle": {
          "title": "Lifecycle", "type": "string",
          "enum": ["production", "beta", "deprecated"],
          "enumColors": { "production": "green", "beta": "yellow", "deprecated": "lightGray" }
        }
      },
      "required": ["classification"]
    },
    "calculationProperties": {},
    "aggregationProperties": {},
    "mirrorProperties": {},
    "relations": {
      "exposed_by": { "title": "Exposed By", "target": "service", "required": false, "many": false }
    },
    "ownership": { "type": "Direct", "title": "Owning Teams" }
  }
}
```

> `ownership: Direct` lets an API carry its own owning team even when it differs
> from the exposing service's team.

---

## 1b. `ikea_permission` (the revocable edge)

```
CallMcpTool server="user-port-eu" toolName="upsert_blueprint" arguments=
```
```json
{
  "blueprint": {
    "identifier": "ikea_permission",
    "title": "IKEA Permission",
    "icon": "Lock",
    "schema": {
      "properties": {
        "scope": {
          "title": "Scope", "type": "string",
          "enum": ["read", "write", "admin"],
          "enumColors": { "read": "green", "write": "orange", "admin": "red" }
        },
        "status": {
          "title": "Status", "type": "string",
          "enum": ["active", "pending_revocation", "revoked"],
          "enumColors": { "active": "green", "pending_revocation": "yellow", "revoked": "red" }
        },
        "risk_level": {
          "title": "Risk Level", "type": "string",
          "enum": ["low", "medium", "high", "critical"],
          "enumColors": { "low": "green", "medium": "yellow", "high": "orange", "critical": "red" }
        },
        "justification": { "title": "Justification", "type": "string" },
        "granted_at": { "title": "Granted At", "type": "string", "format": "date-time" },
        "last_reviewed_at": { "title": "Last Reviewed At", "type": "string", "format": "date-time" }
      },
      "required": ["scope", "status"]
    },
    "calculationProperties": {},
    "aggregationProperties": {},
    "mirrorProperties": {},
    "relations": {
      "consumer": { "title": "Consumer Service", "target": "service", "required": true, "many": false },
      "target_api": { "title": "Target API", "target": "ikea_api", "required": true, "many": false }
    },
    "ownership": { "type": "Inherited", "title": "Owning Teams", "path": "target_api" }
  }
}
```

> The permission inherits ownership from the **target API** — so approval routing
> (Step 4) lands on the team that owns the thing being accessed.

---

## 1c. `ikea_vuln_finding`

```
CallMcpTool server="user-port-eu" toolName="upsert_blueprint" arguments=
```
```json
{
  "blueprint": {
    "identifier": "ikea_vuln_finding",
    "title": "IKEA Vuln Finding",
    "icon": "Alert",
    "schema": {
      "properties": {
        "cve": { "title": "CVE", "type": "string" },
        "severity": {
          "title": "Severity", "type": "string",
          "enum": ["low", "medium", "high", "critical"],
          "enumColors": { "low": "green", "medium": "yellow", "high": "orange", "critical": "red" }
        },
        "cvss_score": { "title": "CVSS Score", "type": "number" },
        "status": {
          "title": "Status", "type": "string",
          "enum": ["open", "in_progress", "resolved", "accepted_risk"],
          "enumColors": { "open": "red", "in_progress": "orange", "resolved": "green", "accepted_risk": "lightGray" }
        },
        "package": { "title": "Affected Package", "type": "string" },
        "description": { "title": "Description", "type": "string", "format": "markdown" }
      },
      "required": ["severity", "status"]
    },
    "calculationProperties": {},
    "aggregationProperties": {},
    "mirrorProperties": {},
    "relations": {
      "affected_service": { "title": "Affected Service", "target": "service", "required": false, "many": false },
      "affected_api": { "title": "Affected API", "target": "ikea_api", "required": false, "many": false }
    }
  }
}
```

---

## Verify

```
CallMcpTool server="user-port-eu" toolName="list_blueprints" arguments={ "identifiers": ["ikea_api", "ikea_permission", "ikea_vuln_finding"] }
```

**Checkpoint:** All three return full details; `ikea_permission` shows
`consumer → service` and `target_api → ikea_api`.

- UI: Builder → Data model → confirm the three blueprints + relations.

**Pause:** "Catalog is live. Ready to seed the graph (Step 2)?"

---

## Manual fallback (Port UI)

Builder → Data model → **+ Blueprint** → **Edit JSON** → paste the `blueprint`
object from each block above (without the outer `{ "blueprint": ... }` wrapper —
paste the inner object). Create `ikea_api` before the other two.
