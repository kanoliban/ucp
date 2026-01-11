# Universal Care Protocol (Care Index) - Domain Mapping

## Overview

This document maps Universal Commerce Protocol (UCP) concepts to Universal Care Protocol (Care Index) equivalents. The architectural patterns remain identical; only the domain semantics change.

---

## Core Domain Translation

| UCP (Commerce) | Care Index (Care) | Notes |
|----------------|-------------------|-------|
| **Shopping** | **Care** | Top-level domain |
| **Checkout** | **Match** | The core transaction flow |
| **Payment** | **Connection** | How value flows between parties |
| **Fulfillment** | **Service Delivery** | How the service gets performed |
| **Order** | **Care Session** | The unit of completed work |
| **Cart** | **Need Expression** | What the user is seeking |
| **Line Item** | **Care Need** | Individual requirement |
| **Discount** | **Benefit/Subsidy** | Financial assistance programs |

---

## Participant Role Translation

| UCP Role | Care Index Role | Description |
|----------|-----------------|-------------|
| **Platform** | **Care Platform** | Consumer-facing surface (CAN Help Desk, CareSupport app) |
| **Business** | **Care Resource** | Entity providing care services (agency, caregiver, program) |
| **Credential Provider** | **Identity Provider** | Manages caregiver credentials, background checks |
| **Payment Service Provider** | **Benefits Processor** | Handles Medicaid, insurance, program payments |
| **Buyer** | **Care Seeker** | Person/family seeking care |

---

## Capability Translation

### Core Capabilities

| UCP Capability | Care Index Capability | Purpose |
|----------------|----------------------|---------|
| `dev.ucp.shopping.checkout` | `dev.careindex.care.match` | Find and connect with resources |
| `dev.ucp.shopping.order` | `dev.careindex.care.session` | Completed care sessions |
| `dev.ucp.shopping.identity_linking` | `dev.careindex.care.identity` | OAuth for acting on behalf |
| `dev.ucp.shopping.payment_token_exchange` | `dev.careindex.care.benefits_exchange` | Program benefit verification |

### Extensions

| UCP Extension | Care Index Extension | Purpose |
|---------------|---------------------|---------|
| `dev.ucp.shopping.discount` | `dev.careindex.care.benefit` | Medicaid, insurance, subsidies |
| `dev.ucp.shopping.fulfillment` | `dev.careindex.care.delivery` | How/when/where care happens |
| `dev.ucp.shopping.buyer_consent` | `dev.careindex.care.consent` | HIPAA, family consent, POA |

---

## Type Translation

### Core Types

| UCP Type | Care Index Type | Description |
|----------|-----------------|-------------|
| `line_item` | `care_need` | Single care requirement |
| `total` | `care_plan_summary` | Aggregated needs and coverage |
| `postal_address` | `service_location` | Where care is delivered |
| `buyer` | `care_seeker` | Family/individual seeking care |
| `item` | `resource` | Available care resource |

### Fulfillment Types

| UCP Type | Care Index Type | Description |
|----------|-----------------|-------------|
| `fulfillment` | `service_delivery` | How care is provided |
| `fulfillment_method` | `delivery_method` | In-home, facility, virtual |
| `fulfillment_option` | `availability_window` | When resource can provide care |
| `fulfillment_event` | `session_event` | Session start, end, incident |
| `fulfillment_destination` | `care_location` | Where care happens |

### Payment Types

| UCP Type | Care Index Type | Description |
|----------|-----------------|-------------|
| `payment_handler` | `benefit_handler` | How programs/insurance work |
| `payment_instrument` | `payment_source` | Medicaid, private pay, insurance |
| `card_credential` | `benefit_credential` | Program eligibility proof |
| `payment_identity` | `payer_identity` | Who is financially responsible |

---

## State Machine Translation

### UCP Checkout States → Care Index Match States

| UCP State | Care Index State | Description |
|-----------|------------------|-------------|
| `incomplete` | `searching` | Gathering information, finding resources |
| `requires_escalation` | `requires_input` | Human decision needed |
| `ready_for_complete` | `matched` | Resource identified, ready to connect |
| *(new)* | `connected` | Relationship active |
| *(new)* | `in_session` | Care being delivered |

---

## Namespace Ownership

Following UCP's reverse-domain pattern for permissionless extension:

| Domain Owner | Namespace | Example Capabilities |
|--------------|-----------|---------------------|
| Care Index (core) | `dev.careindex.*` | `dev.careindex.care.match` |
| CareSupport | `com.caresupport.*` | `com.caresupport.coordination` |
| Ella Plan | `com.ellaplan.*` | `com.ellaplan.benefits` |
| Zinnia | `com.zinnia.*` | `com.zinnia.wellness` |
| Medicaid Programs | `gov.medicaid.*` | `gov.medicaid.waiver.cdcs` |
| Agencies | `com.{agency}.*` | `com.sunrisehomecare.respite` |

---

## Discovery Profile Translation

### UCP Discovery (`/.well-known/ucp`)

```json
{
  "ucp": {
    "capabilities": ["dev.ucp.shopping.checkout"],
    "extensions": ["dev.ucp.shopping.fulfillment"]
  },
  "payment": {
    "handlers": [...]
  }
}
```

### Care Index Discovery (`/.well-known/care-index`)

```json
{
  "care_index": {
    "capabilities": ["dev.careindex.care.match"],
    "extensions": ["dev.careindex.care.delivery"]
  },
  "benefits": {
    "handlers": [
      {
        "type": "medicaid_waiver",
        "programs": ["MN-CDCS", "MN-EW"],
        "verification_endpoint": "https://..."
      }
    ]
  },
  "service_area": {
    "type": "radius",
    "center": { "lat": 44.9778, "lng": -93.2650 },
    "radius_miles": 25
  },
  "resource_types": ["respite", "overnight", "transportation"],
  "certifications": ["dementia_care", "med_admin"]
}
```

---

## API Endpoint Translation

| UCP Endpoint | Care Index Endpoint | Purpose |
|--------------|---------------------|---------|
| `POST /checkout` | `POST /match` | Initiate matching session |
| `GET /checkout/{id}` | `GET /match/{id}` | Get match status |
| `PATCH /checkout/{id}` | `PATCH /match/{id}` | Update match with info |
| `POST /checkout/{id}/complete` | `POST /match/{id}/connect` | Finalize connection |
| `GET /orders/{id}` | `GET /sessions/{id}` | Get session details |

---

## File Structure Transformation

```
source/
├── schemas/
│   ├── care/                          # Was: shopping/
│   │   ├── match.json                 # Was: checkout.json
│   │   ├── session.json               # Was: order.json
│   │   ├── benefit.json               # Was: discount.json
│   │   ├── delivery.json              # Was: fulfillment.json
│   │   ├── consent.json               # Was: buyer_consent.json
│   │   └── types/
│   │       ├── care_need.json         # Was: line_item.json
│   │       ├── resource.json          # Was: item.json
│   │       ├── care_seeker.json       # Was: buyer.json
│   │       ├── service_location.json  # Was: postal_address.json
│   │       ├── availability.json      # Was: fulfillment_option.json
│   │       ├── session_event.json     # Was: fulfillment_event.json
│   │       └── benefit_handler.json   # Was: payment_handler.json
│   ├── capability.json                # Same structure, new domains
│   └── ucp.json → care_index.json
├── discovery/
│   └── profile_schema.json            # Adapted for care resources
└── bindings/                          # REST, MCP, A2A (same pattern)
```

---

## Implementation Priority

### Phase 1: Core Schema (Week 1)
1. `care_index.json` - Protocol metadata
2. `capability.json` - Adapted for care domains
3. `match.json` - Core matching capability
4. Core types: `resource.json`, `care_need.json`, `care_seeker.json`

### Phase 2: Capabilities (Week 2)
1. `session.json` - Care session lifecycle
2. `delivery.json` - Service delivery extension
3. `benefit.json` - Benefits/subsidy extension
4. `consent.json` - HIPAA/family consent extension

### Phase 3: Discovery & Bindings (Week 3)
1. `profile_schema.json` - Resource discovery
2. REST binding specification
3. MCP binding (for AI agent integration)

---

## Key Differences from UCP

| Aspect | UCP (Commerce) | Care Index (Care) |
|--------|----------------|-------------------|
| **Anonymity** | Buyer can be anonymous | Care seeker identity required (for safety) |
| **Verification** | Payment credentials | Background checks, certifications |
| **Consent** | Purchase consent | HIPAA, POA, family authority |
| **Recurring** | Subscription model | Ongoing care relationship |
| **Matching** | Inventory lookup | Capability + availability + proximity |
| **Quality** | Reviews/ratings | Verified outcomes, incident history |

---

## Open Questions

1. **Credential verification**: How do we verify caregiver credentials without central authority?
2. **HIPAA compliance**: What PHI can flow through the protocol vs. stays out-of-band?
3. **Emergency escalation**: How does `requires_escalation` work for urgent care needs?
4. **Multi-party consent**: How do we handle family POA and shared decision-making?

---

*This document serves as the translation layer between UCP's commerce domain and Care Index's care coordination domain. The architectural patterns (capabilities, extensions, discovery, state machine) remain unchanged.*
