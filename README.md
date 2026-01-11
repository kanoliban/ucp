<!--
   Copyright 2026 Universal Care Protocol Authors

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
-->

<p align="center">
  <h1 align="center">Universal Care Protocol (Care Index)</h1>
</p>

<p align="center">
  <b>An open standard enabling interoperability between care platforms,
   caregivers, agencies, and benefit programs to facilitate seamless
   care coordination.</b>
</p>

<p align="center">
  <a href="https://careindex.dev">Documentation</a> |
  <a href="https://careindex.dev/specification/overview">Specification</a> |
  <a href="https://github.com/kanoliban/ucp/discussions">Discussions</a>
</p>

## Overview

The Universal Care Protocol (Care Index) addresses a fragmented caregiving landscape
by providing a standardized common language and functional primitives. It
enables care platforms (like CareSupport, CAN Help Desk, AI agents), care resources
(caregivers, agencies, programs), and benefit providers to communicate effectively,
ensuring consistent care coordination experiences.

With Care Index, care resources can:

*   **Declare** supported capabilities to enable autonomous discovery by
    platforms.
*   **Facilitate** care matching sessions, with or without human intervention.
*   **Provide** personalized care recommendations through standardized data
    exchange.

## Why Care Index?

As care coordination becomes increasingly complex and distributed, the ability for
different systems to interoperate without custom, one-off integrations is vital.
Care Index aims to:

*   **Standardize Interaction:** Provide a uniform way for platforms to interact
    with care resources, regardless of the underlying backend.
*   **Modularize Care:** Break down care into distinct **Capabilities**
    (e.g., Match, Session) and **Extensions** (e.g., Benefits,
    Delivery), allowing for flexible implementation.
*   **Enable Agentic Care Coordination:** Designed from the ground up to support AI
    agents acting on behalf of families to discover resources, match needs, and
    connect with caregivers.
*   **Enhance Trust:** Support for credential verification, consent management,
    and quality signals.

### Key Features

*   **Composable Architecture:** Care Index defines **Capabilities** (such as
    "Match" or "Session") that resources implement to enable easy
    integration. On top of that, specific **Extensions** can be added to enhance
    the experience without bloating the capability definitions.
*   **Dynamic Discovery:** Resources declare their supported Capabilities in a
    standardized profile at `/.well-known/care-index`, allowing platforms to
    autonomously discover and configure themselves.
*   **Transport Agnostic:** The protocol is designed to work across various
    transports. Resources can offer Capabilities via REST APIs, MCP (Model
    Context Protocol), or A2A, depending on their infrastructure.
*   **Built on Standards:** Care Index leverages existing open standards for
    identity, security, and healthcare interoperability wherever applicable.
*   **Permission-Free Extension:** Uses reverse-domain naming for namespaces,
    allowing any organization to extend the protocol without central approval.

## Key Capabilities

The initial release focuses on the essential primitives for care coordination:

*   **Match:** Facilitates care matching sessions including need expression,
    resource discovery, and availability checking—supporting flows with or without
    human intervention.
*   **Session:** Webhook-based updates for care session lifecycle events (started,
    completed, incident reported).
*   **Identity:** Enables platforms to obtain authorization to perform
    actions on a care seeker's behalf via OAuth 2.0.
*   **Benefits Exchange:** Protocols for verifying and applying benefit program
    eligibility (Medicaid waivers, insurance, subsidies).

## Domain Translation from UCP

Care Index is architecturally identical to the Universal Commerce Protocol (UCP),
adapted for the care coordination domain:

| UCP (Commerce) | Care Index (Care) |
|----------------|-------------------|
| Shopping | Care |
| Checkout | Match |
| Payment | Connection |
| Fulfillment | Service Delivery |
| Order | Care Session |
| Business | Care Resource |
| Buyer | Care Seeker |

See [CARE_INDEX_MAPPING.md](CARE_INDEX_MAPPING.md) for complete domain translation.

## Namespace Ownership

Following UCP's reverse-domain pattern for permissionless extension:

| Domain Owner | Namespace | Examples |
|--------------|-----------|----------|
| Care Index (core) | `dev.careindex.*` | `dev.careindex.care.match` |
| CareSupport | `com.caresupport.*` | `com.caresupport.coordination` |
| Caregiver Action Network | `org.can.*` | `org.can.helpdesk` |
| Ella Plan | `com.ellaplan.*` | `com.ellaplan.benefits` |
| Medicaid Programs | `gov.medicaid.*` | `gov.medicaid.waiver.cdcs` |

## Getting Started

*   📚 **Explore the Documentation:** Visit [careindex.dev](https://careindex.dev) for a
    complete overview, the full protocol specification, tutorials, and guides.
*   🎬 **Review our samples** for implementation examples.
*   🛠️ **Use our SDKs** to start building your own integrations.

## Project Structure

```
source/
├── schemas/
│   ├── care/                    # Care-specific schemas
│   │   ├── match.json           # Core matching capability
│   │   ├── session.json         # Care session lifecycle
│   │   └── types/               # Shared type definitions
│   │       ├── resource.json    # Care resource schema
│   │       ├── care_need.json   # Care need expression
│   │       ├── care_seeker.json # Family/individual seeking care
│   │       ├── availability.json
│   │       └── service_area.json
│   ├── care_index.json          # Protocol metadata
│   └── capability.json          # Capability definitions
├── discovery/
│   └── profile_schema.json      # Discovery profile at /.well-known/care-index
└── bindings/                    # Transport bindings (REST, MCP, A2A)
```

## Contributing

We welcome community contributions to enhance and evolve Care Index.

*   **Questions & Discussions:** Join our GitHub Discussions.
*   **Issues & Feedback:** Report issues or suggest improvements via GitHub Issues.
*   **Contribution Guide:** See our [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## What's Next

Future enhancements include:

*   **New Care Types:** Applications beyond home care (e.g., facility care, respite programs).
*   **Quality Signals:** Standardized quality metrics and outcome tracking.
*   **Care Context Graph:** Integration with CareSupport's decision trace infrastructure.

## Relationship to CareSupport

Care Index is the open specification. CareSupport operates the reference implementation.

- **Specification (open):** This repository—anyone can implement
- **Implementation (operated):** CareSupport runs the matching infrastructure
- **Data (contributed):** Resources contribute availability; platforms contribute needs
- **Distribution (CAN):** Caregiver Action Network provides access to 40K caregivers

This follows the pattern established by UCP: operators build the standard, not committees.

## About

Care Index is an open-source project under the [Apache License 2.0](LICENSE) and is
open to contributions from the community.

*Forked from [Universal Commerce Protocol](https://github.com/Universal-Commerce-Protocol/ucp)
with domain adaptation for care coordination.*
