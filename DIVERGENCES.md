# Care Index Divergences from UCP

**Version:** 2026-01-11
**Status:** Intentional departures documented

Care Index is forked from the Universal Commerce Protocol (UCP). While the architectural principles are isomorphic, care coordination and commerce have different risk profiles. This document captures **intentional divergences** from UCP and the reasoning behind each.

---

## Principle: Build on UCP's Spirit, Not Its Letter

> "Take the principles (discovery, negotiation, escalation, decentralized governance). Adapt the practices (versioning, caching, schema strictness, fallback strategy) to match care's risk profile."

---

## Divergence 1: Strict Version Matching

### UCP Approach
```
IF platform_version ≤ business_version THEN proceed
```

### Care Index Approach
```
IF platform_version == resource_version THEN proceed
ELSE error: "Version mismatch. Please update."
```

### Why This Matters

**UCP context:** Commerce is transactional. A 2024-era platform talking to a 2026-era business completes checkout, money moves, goods ship. If semantics drift slightly, the transaction still completes.

**Care context:** Care is continuous. A platform and resource on different versions might agree on shift scheduling but disagree on handoff semantics. The shift "completes" but the handoff fails silently. A person doesn't receive medication reminders. Harm occurs.

**Care Index rule:** Version mismatch MUST error immediately. No permissive fallback. The cost of explicit failure is a coordination delay. The cost of silent drift is harm to people.

### Schema Reference
```json
{
  "version_negotiation": {
    "required": ["platform_version", "resource_version", "match_status"],
    "properties": {
      "match_status": {
        "enum": ["exact_match", "version_mismatch"]
      }
    }
  }
}
```

---

## Divergence 2: Mandatory Cache Limits

### UCP Approach
```
Platforms SHOULD cache profiles per HTTP directives
```

### Care Index Approach
```
Platforms MUST revalidate profiles before matching
Maximum cache age: 4 hours for profiles, 1 hour for availability
```

### Why This Matters

**UCP context:** A merchant's checkout capabilities change rarely. A 24-hour stale cache might mean a discount is no longer valid. Minor friction.

**Care context:** Caregiver availability changes hourly. Jennifer is available at 2pm when you cached the profile; she's booked when you propose the match 6 hours later. You've promised care that doesn't exist. Family is frustrated. Trust erodes.

**Care Index rule:**
- Profile cache: max 4 hours (`max_age_seconds: 14400`)
- Availability cache: max 1 hour (`availability_max_age_seconds: 3600`)
- `must_revalidate: true` — always check before matching

### Schema Reference
```json
{
  "cache_requirements": {
    "required": ["max_age_seconds", "must_revalidate"],
    "properties": {
      "max_age_seconds": {
        "maximum": 14400
      },
      "availability_max_age_seconds": {
        "maximum": 3600
      },
      "must_revalidate": {
        "const": true
      }
    }
  }
}
```

---

## Divergence 3: Mandatory Fallback Strategy

### UCP Approach
```
requires_escalation → human completes checkout
(Implicit: PSPs serve as clearing house; payment always has a path)
```

### Care Index Approach
```
requires_input → explicit fallback_strategy required
(Care has no guaranteed clearing house; fallback must be defined)
```

### Why This Matters

**UCP context:** If checkout can't complete, PSPs step in. Stripe, Square, PayPal—there's always a way to move money. The fallback is implicit because financial infrastructure is mature.

**Care context:** If matching fails, what happens? There's no "Stripe for caregivers." Families still need care. The matching algorithm failing doesn't make the care need go away.

**Care Index rule:** Every escalation MUST include a `fallback_strategy` specifying what happens next:
- `human_coordinator` — route to human for manual matching
- `waitlist` — add to queue, notify when available
- `expand_search` — suggest relaxing constraints
- `emergency_backup` — activate backup pool
- `family_network` — fall back to existing network
- `agency_referral` — escalate to agency
- `community_board` — post to community for volunteers

### Schema Reference
```json
{
  "escalation": {
    "required": ["reason", "fallback_strategy"],
    "properties": {
      "fallback_strategy": {
        "type": {
          "enum": [
            "human_coordinator",
            "waitlist",
            "expand_search",
            "emergency_backup",
            "family_network",
            "agency_referral",
            "community_board"
          ]
        }
      }
    }
  }
}
```

---

## Divergence 4: Transparent Governance

### UCP Approach
```
Frames all parties as equal
(Obscures: Google drives consumer adoption; Shopify drives merchant adoption)
```

### Care Index Approach
```
Explicit about power asymmetries
Specification steward: CAN
Reference operator: CareSupport
Extension governance: permissionless namespace
```

### Why This Matters

**UCP context:** Commercial participants understand the power dynamics implicitly. Everyone knows Google controls the consumer surface. The marketing fiction of equality is acceptable because participants are sophisticated.

**Care context:** Care stakeholders include families, individual caregivers, small agencies, and government programs. Power asymmetries that are obvious to tech insiders are not obvious to care stakeholders. Obscuring governance breeds mistrust.

**Care Index rule:** The specification explicitly states:
- **CAN** stewards the open specification (governance)
- **CareSupport** operates the reference implementation (execution)
- **Anyone** can extend via reverse-domain namespace (ecosystem)

This isn't equal. CareSupport has operational leverage. Own it.

### Schema Reference
```json
{
  "governance": {
    "properties": {
      "specification_steward": {
        "const": "Caregiver Action Network (CAN)"
      },
      "reference_operator": {
        "const": "CareSupport"
      },
      "extension_governance": {
        "const": "permissionless_namespace"
      }
    }
  }
}
```

---

## Divergence 5: Strict Schemas for Safety-Critical Fields

### UCP Approach
```
Risk signals have "flexible structure"
(Fraud prevention varies by platform)
```

### Care Index Approach
```
Safety-critical fields have strict schemas
(Caregiver verification, quality signals, incident reporting)
```

### Why This Matters

**UCP context:** Fraud signals are platform-specific. Stripe has different fraud detection than Square. "Flexible structure" allows each to contribute what they know. Interoperability suffers, but fraud prevention is platform-specific anyway.

**Care context:** Caregiver verification is life-safety. If verification schemas are flexible, one platform accepts "self-attested" while another requires "issuer_verified." Families can't compare. Trust erodes. Bad actors exploit gaps.

**Care Index rule:** Safety-critical fields have strict schemas:
- `quality_signals.verification.identity_verified` — boolean, not flexible
- `quality_signals.verification.background_check_current` — boolean, not flexible
- `session.incidents[].severity` — enum `[low, medium, high, critical]`, not flexible

### Schema Reference
```json
{
  "quality_signals": {
    "verification": {
      "properties": {
        "identity_verified": {"type": "boolean"},
        "background_check_current": {"type": "boolean"},
        "credentials_verified": {"type": "boolean"}
      }
    }
  }
}
```

---

## Divergence 6: Namespace Validation is MUST, not SHOULD

### UCP Approach
```
Platform SHOULD validate namespace binding
(Optional validation; if skipped, security model weakens)
```

### Care Index Approach
```
Platform MUST validate namespace binding
Validation failure MUST error
(Non-negotiable for care coordination)
```

### Why This Matters

**UCP context:** If 10% of platforms skip namespace validation for speed, the ecosystem has fragmented security. But commerce has layers of protection (chargebacks, PSP fraud detection, legal recourse).

**Care context:** If a malicious actor spoofs `gov.medicaid.waiver.cdcs`, families might believe they have coverage they don't. Or worse, share PHI with an unauthorized party. There's no chargeback for privacy violations.

**Care Index rule:** Namespace validation is mandatory:
1. Spec URL MUST originate from declared namespace authority
2. Validation failure MUST error immediately
3. Platforms MUST NOT proceed with unvalidated extensions

---

## Summary: The Five Questions

Before implementing any feature, ask:

| Question | UCP Answer | Care Index Answer |
|----------|------------|-------------------|
| What if versions mismatch? | Proceed (permissive) | Error (strict) |
| How long can I cache? | Per HTTP directives | Max 4h profile, 1h availability |
| What if no match? | Implicit PSP fallback | Explicit fallback_strategy required |
| Who controls governance? | Obscured | Explicit: CAN stewards, CareSupport operates |
| How strict are safety fields? | Flexible | Strict schemas, no flexibility |

---

## Changelog

- **2026-01-11:** Initial divergences documented based on critical analysis of UCP v2026-01-11
