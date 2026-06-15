# Step 3 — Access-Hygiene Scorecard

**Goal:** Score every permission for least-privilege hygiene so security can spot
risky grants *before* a change is proposed.

**Blueprint:** `ikea_permission`
**Scorecard:** `ikea_access_hygiene`

## Levels & rules

- **Basic** (default, no rules — every permission starts here).
- **Bronze** — has a justification (no orphan grants).
- **Silver** — Bronze + not `admin` scope (least privilege) OR reviewed recently.
- **Gold** — Silver + risk_level is `low` or `medium` and status `active`.

> Rules can't be attached to `Basic` (Port returns 409). Only Bronze/Silver/Gold.

```
CallMcpTool server="user-port-eu" toolName="upsert_scorecard" arguments=
```
```json
{
  "blueprintIdentifier": "ikea_permission",
  "scorecard": {
    "identifier": "ikea_access_hygiene",
    "title": "Access Hygiene",
    "levels": [
      { "title": "Basic", "color": "lightGray" },
      { "title": "Bronze", "color": "bronze" },
      { "title": "Silver", "color": "silver" },
      { "title": "Gold", "color": "gold" }
    ],
    "rules": [
      {
        "identifier": "has_justification",
        "title": "Has documented justification",
        "level": "Bronze",
        "query": {
          "combinator": "and",
          "conditions": [ { "property": "justification", "operator": "isNotEmpty" } ]
        }
      },
      {
        "identifier": "least_privilege",
        "title": "Not admin scope",
        "level": "Silver",
        "query": {
          "combinator": "and",
          "conditions": [ { "property": "scope", "operator": "!=", "value": "admin" } ]
        }
      },
      {
        "identifier": "low_risk_active",
        "title": "Low/medium risk and active",
        "level": "Gold",
        "query": {
          "combinator": "and",
          "conditions": [
            { "property": "risk_level", "operator": "in", "value": ["low", "medium"] },
            { "property": "status", "operator": "=", "value": "active" }
          ]
        }
      }
    ]
  }
}
```

## Verify

```
CallMcpTool server="user-port-eu" toolName="list_scorecards" arguments={ "blueprintIdentifier": "ikea_permission" }
```

**Checkpoint:** Each seeded permission now carries a level. The high-risk
`ikea-perm-checkout-to-payments` (write) should sit below Gold — a visible
signal it deserves scrutiny.

**Pause:** "Hygiene scoring is on. Ready to build the revoke action (Step 4)?"

## Manual fallback

Builder → `ikea_permission` blueprint → **Scorecards** tab → **+ Scorecard** →
Edit JSON → paste the `scorecard` object above.
