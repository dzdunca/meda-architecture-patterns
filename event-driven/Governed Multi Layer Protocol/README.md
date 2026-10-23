---
id: governed-multi-protocol-integration-layer
---

# Pattern – Governed Multi-Protocol Integration Layer

## 1. Introduction

Pattern 4 defines the **Governed Multi-Protocol Integration Layer** that enables Kafka-native applications and systems using alternative integration protocols to securely publish events into the TMR MEDA Event Backbone.

The pattern extends the zero-trust and Identity Firewall principles established in Pattern 3 by introducing a governed ingestion architecture that supports:

- Native Kafka event streaming
- AWS EventBridge event integration
- File-based ingestion
- Database ingestion and Change Data Capture (CDC)
- Object storage ingestion
- Legacy batch integration
- HTTP-based ingestion
- Secure external partner onboarding

The pattern establishes two supported ingestion lanes:

1. **Lane 1 – Kafka-Native Ingestion:** Applications publish directly to governed Kafka topics using approved Kafka protocols, identities and schemas.
2. **Lane 2 – Platform-Mediated Ingestion:** Applications and external systems publish through approved integration services, including AWS EventBridge, managed connectors, ingestion APIs and controlled landing zones.

Both lanes ultimately publish into the MEDA Event Backbone under consistent enterprise governance requirements.

The pattern ensures that protocol diversity does not require uncontrolled Kafka broker access, weaken identity governance or compromise the integrity of trusted event streams.

**Architectural principle:** The source integration protocol may vary, but all data entering trusted MEDA event streams must satisfy the applicable identity, schema, security, lineage and operational governance requirements.

---

## 2. Purpose and Architectural Outcomes

The TMR integration landscape includes modern event-native applications, AWS-native services, enterprise applications, legacy platforms and external partners.

Not all producers can or should connect directly to Kafka.

For example:

- A microservice may already publish business events through Amazon EventBridge.
- A legacy application may produce scheduled CSV exports.
- An operational database may require CDC.
- An external partner may deliver files through SFTP.
- An enterprise application may expose data through an HTTP API.
- A Kafka-native service may publish directly using the Kafka protocol.

Mandating Kafka-native integration for every producer would increase delivery complexity, introduce unnecessary coupling and create additional modernisation costs.

Pattern 4 addresses this by providing governed integration pathways that allow systems to use appropriate source protocols while preserving Kafka as the canonical MEDA Event Backbone.

### 2.1 Architectural Outcomes

The pattern provides:

- A consistent governance model across multiple ingestion mechanisms.
- Secure onboarding for internal and external event producers.
- Isolation between external systems and internal Kafka infrastructure.
- Controlled translation of non-Kafka protocols into Kafka event streams.
- Schema governance and validation before trusted consumption.
- Source identity preservation and data lineage.
- Operational isolation between ingestion mechanisms.
- Controlled replay, error handling and recovery.
- Progressive modernisation without requiring immediate replacement of legacy integration mechanisms.

### 2.2 Core Architectural Decisions

The following decisions govern implementations of this pattern:

| Decision | Architectural requirement |
|---|---|
| Canonical event backbone | Kafka is the canonical event streaming backbone for MEDA. |
| Kafka-native producers | Approved applications may connect directly using governed Kafka identities and access controls. |
| Platform-mediated producers | Non-Kafka producers use approved integration services rather than unmanaged direct broker access. |
| EventBridge integration | AWS EventBridge is an approved logical ingestion pathway, subject to implementation and security approval. |
| Trusted publication | Records must satisfy applicable validation and governance controls before entering trusted topics. |
| Identity isolation | Source identities and Kafka publishing identities remain independently governed. |
| Source provenance | The originating application, system or partner must remain identifiable. |
| Payload management | Large payloads should be stored outside Kafka where appropriate, with events carrying references and metadata. |
| Operational resilience | Ingestion implementations must define delivery guarantees, retries, duplicate handling and recovery. |

---

## 3. When to Use This Pattern

Use this pattern when an application, enterprise system or external partner must publish events or source data into the MEDA Event Backbone.

Typical scenarios include:

- Kafka-native microservices publishing business events.
- AWS applications publishing events to Amazon EventBridge.
- Legacy systems exporting data through SFTP.
- Applications placing files into S3 landing buckets.
- Database replication and CDC.
- Scheduled batch exports requiring transformation into event streams.
- External partner integrations requiring controlled ingress.
- HTTP-based producers requiring an approved publishing gateway.

### 3.1 When Not to Use This Pattern

This pattern should not be used to introduce unnecessary integration components when a simpler approved integration path already exists.

For example:

- A Kafka-native application should not be required to publish through EventBridge merely to reach Kafka.
- A system should not require an additional managed connector if an existing approved ingestion capability already satisfies the requirements.
- Large documents and binary objects should not be embedded in Kafka events when object storage and references are more appropriate.
- A request-response API integration should not automatically be converted into event streaming unless there is a genuine event-driven requirement.

### 3.2 Selection Guidance

| Producer capability | Preferred ingestion approach |
|---|---|
| Native Kafka producer | Lane 1 – Kafka-Native Ingestion |
| AWS-native event publisher | Lane 2 – EventBridge-mediated ingestion |
| S3 object producer | Lane 2 – Object storage ingestion |
| SFTP or batch file producer | Lane 2 – File ingestion |
| Relational database source | Lane 2 – JDBC or CDC ingestion |
| HTTP-only application | Lane 2 – Governed API or HTTP ingestion |
| External partner | Lane 2 – Approved partner ingress boundary |

The selected mechanism must be validated against security, throughput, latency, reliability, data classification and operational requirements.

---

## 4. Solution Architecture

### 4.1 Dual-Lane Ingestion Model

The architecture separates direct Kafka publication from platform-mediated ingestion.

```text
                        EVENT PRODUCERS
                               |
             +-----------------+------------------+
             |                                    |
             v                                    v
     LANE 1 - KAFKA NATIVE              LANE 2 - PLATFORM MEDIATED
             |                                    |
     Kafka-Native Applications          +---------+----------+
             |                          |         |          |
             |                          v         v          v
             |                      EventBridge   S3       SFTP/CDC
             |                          |         |          |
             |                          v         v          v
             |                      Kafka Bridge / Managed Ingestion
             |                                    |
             |                                    v
             |                              Raw Ingress Topics
             |                                    |
             |                                    v
             |                         Validation / Transformation
             |                                    |
             +------------------+-----------------+
                                |
                                v
                    TRUSTED MEDA KAFKA TOPICS
                                |
                   +------------+------------+
                   |            |            |
                   v            v            v
               Consumers    Analytics    Persistence
```

This is a logical architecture. The exact processing components and the need for intermediate raw topics depend on the selected ingestion mechanism and its governance requirements.

### 4.2 Lane 1 – Kafka-Native Ingestion

Kafka-native applications publish directly to approved Kafka topics using supported Kafka client protocols.

These producers must comply with:

- Approved authentication mechanisms.
- Kafka ACL and RBAC requirements.
- Schema Registry governance.
- Topic naming and ownership standards.
- Network connectivity controls.
- Producer reliability requirements.
- Audit and observability requirements.

Kafka-native publication does not automatically imply that the data is trusted.

The topic classification, configured validation controls and producer contract determine whether the published records are suitable for trusted consumption.

### 4.3 Lane 2 – Platform-Mediated Ingestion

Platform-mediated ingestion supports producers that cannot or should not communicate directly with Kafka.

Supported logical integration mechanisms include:

- AWS EventBridge
- Amazon S3 landing zones
- Secure FTP/SFTP ingestion
- JDBC ingestion
- Change Data Capture
- Governed HTTP ingestion
- Managed Kafka Connect source connectors
- Approved ingestion applications or workers

These services terminate the source protocol and publish into Kafka using centrally governed integration identities.

Platform-mediated ingestion may require intermediate raw topics, transformation services or validation pipelines before records are promoted into trusted event streams.

### 4.4 Governance Consistency

Both lanes must provide:

- Authorised source access.
- Controlled Kafka publication.
- Approved data contracts.
- Source provenance.
- Appropriate schema validation.
- Operational monitoring.
- Error handling and recovery.
- Auditable governance decisions.

The implementation mechanisms may differ, but the required governance outcomes remain consistent.

---

## 5. Architectural Governance and Tenancy

### 5.1 Streaming and Processing Plane – Confluent Cloud

Confluent Cloud provides the managed streaming and processing capabilities used by the MEDA Event Backbone.

Depending on the approved solution, these may include:

- Kafka clusters and topics.
- Schema Registry.
- Flink stream processing.
- Managed Kafka Connect connectors.
- Service accounts.
- ACLs and RBAC.
- Streaming observability and operational controls.

These resources are governed by the MEDA platform team.

Use of a particular managed connector or processing service is subject to capability availability, security review and operational approval.

### 5.2 Landing, Persistence and Enterprise Control Plane – MEDA AWS

The MEDA AWS environment provides enterprise-controlled integration and persistence capabilities.

These may include:

- S3 landing buckets.
- Amazon EventBridge event buses.
- AWS Lambda ingestion or publishing services.
- API Gateway.
- AWS Secrets Manager.
- IAM roles and policies.
- RDS or Aurora databases.
- Audit storage.
- Data lineage and operational services.

Not every implementation requires every component.

For example, an EventBridge-to-Kafka integration may not require an S3 landing bucket, while a file ingestion solution may not require EventBridge.

### 5.3 Integration and Protocol Boundary

The integration boundary separates source systems from the MEDA Event Backbone.

Its responsibilities include:

- Source authentication and authorisation.
- Protocol termination.
- Secure credential management.
- Controlled Kafka publication.
- Transformation where required.
- Error handling.
- Operational telemetry.
- Preservation of source identity and lineage.

External producers must not be granted access to MEDA infrastructure beyond their approved integration interfaces.

---

## 6. Multi-Protocol Ingestion Configuration

| Capability | Lane 1 – Kafka-Native | Lane 2 – Platform-Mediated |
|---|---|---|
| Source interface | Kafka protocol | EventBridge, S3, SFTP, JDBC, CDC, HTTP |
| Authentication | Approved Kafka authentication | Source-specific authentication and IAM |
| Kafka broker access | Direct, governed | Through approved publishing component |
| Kafka publishing identity | Producer service account | Connector or bridge service account |
| Payload formats | Approved event schemas | JSON, CSV, XML, database rows, files |
| Schema validation | Producer and/or broker controls | Connector, bridge or transformation pipeline |
| Processing | Streaming | Streaming, event-driven, polling or batch |
| Delivery | Kafka producer semantics | Source-specific delivery and bridge semantics |
| Failure handling | Producer retries and governed error paths | Retry, quarantine, DLQ and replay |
| Source provenance | Producer identity and event metadata | Original source identity and integration metadata |
| Trust boundary | Producer-to-Kafka | Source-to-platform integration boundary |

---

## 7. AWS EventBridge to Kafka Integration

### 7.1 Purpose

AWS EventBridge provides a platform-mediated ingestion mechanism for AWS applications and services that already publish events using the EventBridge event model.

This enables AWS-native producers to participate in the MEDA Event Backbone without requiring each application to implement a Kafka producer.

### 7.2 Logical Architecture

```text
AWS Application / Microservice
              |
              v
      Amazon EventBridge
              |
              v
     Governed Kafka Bridge
              |
              v
       Kafka Ingress Topic
              |
              v
    Validation / Transformation
         (where required)
              |
              v
      Trusted Kafka Topic
              |
              v
       MEDA Consumers
```

The Kafka bridge is a logical capability and may be implemented using an approved integration mechanism.

Potential implementation options include:

- EventBridge API Destinations targeting an approved HTTPS publishing endpoint.
- An AWS Lambda function implementing a governed Kafka producer.
- An approved integration service that accepts EventBridge events and publishes to Kafka.

The selected implementation must satisfy connectivity, authentication, delivery and operational requirements.

An EventBridge API Destination must not be assumed to support the Kafka wire protocol directly. A compatible HTTPS endpoint or publishing service is required.

### 7.3 EventBridge Governance

The EventBridge integration must define:

- Approved source event buses.
- Event rules and routing criteria.
- Event producer ownership.
- IAM permissions.
- Kafka publishing identities.
- Event-to-Kafka topic mappings.
- Event schema validation.
- Retry and dead-letter handling.
- Monitoring and alerting.
- Source provenance.

### 7.4 Identity Separation

An AWS application publishes using its approved AWS identity.

The Kafka bridge publishes using a separately governed Kafka identity.

These identities must not be treated as interchangeable.

For example:

```text
AWS Application IAM Role
            |
            v
      EventBridge
            |
            v
    Kafka Bridge IAM Role
            |
            v
   Kafka Service Account
            |
            v
       Kafka Topic
```

The original AWS event source must remain identifiable in the published event metadata.

### 7.5 EventBridge Delivery Considerations

The solution must account for:

- At-least-once event delivery and possible duplicates.
- Retry behaviour.
- Event ordering limitations.
- Event size limits.
- Downstream Kafka publication failures.
- EventBridge DLQ configuration.
- Idempotent event processing.
- Event correlation and traceability.

EventBridge should not be assumed to provide Kafka partition ordering or end-to-end exactly-once delivery.

### 7.6 EventBridge Use Cases

Typical use cases include:

- AWS microservices publishing domain events.
- AWS platform events requiring enterprise distribution.
- Application events already routed through an enterprise event bus.
- Event-driven integration between AWS applications and MEDA consumers.

---

## 8. Connectivity and Infrastructure – Controlled Intake Model

### 8.1 Kafka-Native Producer Connectivity

Kafka-native producers connect through approved Kafka ingress endpoints.

Security controls may include:

- TLS encryption.
- Supported SASL or OAuth-based authentication.
- mTLS where supported by the selected service and connectivity model.
- Private connectivity or approved network restrictions.
- Kafka ACLs.
- RBAC.
- Approved service accounts.

The specific authentication mechanism must be supported by the target Kafka platform.

Enterprise identity federation does not automatically imply that Kafka clients can authenticate using the same identity mechanism.

### 8.2 Platform-Mediated Connectivity

Non-Kafka producers connect only through approved integration endpoints.

Examples include:

- S3 landing buckets.
- Secure SFTP services.
- JDBC or CDC connections.
- EventBridge event buses.
- Governed HTTP endpoints.
- Approved ingestion applications.

The ingestion service performs protocol translation and Kafka publication under a controlled identity.

### 8.3 Network Isolation

Implementations must ensure:

- External systems cannot enumerate internal Kafka infrastructure.
- Kafka credentials are not unnecessarily distributed to external producers.
- Ingestion components use approved network paths.
- Inbound access is restricted to required interfaces.
- Secrets are managed through approved secret-management services.
- Network and identity boundaries are documented.

---

## 9. Heavy Path and Light Path – Translation Boundary

### 9.1 Heavy Path – Source Transport

Large payloads and non-streaming workloads are handled through appropriate integration mechanisms.

Examples include:

- S3 object storage.
- SFTP transfers.
- Database snapshots.
- JDBC polling.
- CDC extraction.
- Batch exports.

These mechanisms allow the platform to control source ingestion independently of Kafka processing.

### 9.2 Light Path – Event Promotion

Where appropriate, the ingestion process publishes lightweight events into Kafka.

These events may contain:

- Business identifiers.
- Event type.
- Source system.
- Event timestamp.
- Correlation identifiers.
- Schema information.
- Lineage metadata.
- References to externally stored payloads.

For example, a large source document may be stored in S3 while Kafka carries a governed event containing its S3 reference.

### 9.3 Architectural Rule

Kafka should carry event data appropriate to the streaming platform.

Large binary objects, unmanaged bulk files and oversized source payloads should not be embedded in Kafka messages when an external storage reference is more appropriate.

---

## 10. Identity and Access Governance – Identity Firewall

Pattern 4 extends the Identity Firewall established in Pattern 3.

### 10.1 Kafka-Native Producer Identity

Kafka-native applications authenticate using approved Kafka identities.

Kafka permissions must be restricted to the required operations and resources.

Example:

```text
ALLOW Principal:User:vehicle-ai-engine
Operation: WRITE
Resource: Topic:vehicle.telemetry.raw
```

Actual permissions must be implemented using the approved Kafka authorisation model.

### 10.2 Platform-Mediated Producer Identity

Platform-mediated ingestion separates source authentication from Kafka publication.

For example:

```text
External Partner
       |
       | Partner Authentication
       v
Governed SFTP Landing
       |
       | Ingestion Service Identity
       v
Kafka Source Connector
       |
       | Kafka Service Account
       v
Kafka Ingress Topic
```

The external partner does not require Kafka credentials.

### 10.3 Source Identity and Provenance

A connector service account identifies the component publishing to Kafka.

It does not necessarily identify the original system that supplied the data.

The ingestion process must preserve sufficient metadata to identify:

- Original source system.
- Source application or partner.
- Ingestion mechanism.
- Event or file identifier.
- Relevant source timestamps.
- Correlation or trace identifier.
- Processing timestamp.

### 10.4 Blind Ingress Constraint

External producers using mediated ingestion must not receive unnecessary access to internal streaming infrastructure.

Unless separately authorised, they must not be able to:

- Read Kafka topics.
- Enumerate Kafka brokers.
- Access Schema Registry.
- Join Kafka consumer groups.
- Inspect internal streaming metadata.
- Access unrelated MEDA integration resources.

Their permissions are restricted to the approved intake interface.

---

## 11. Data Integrity and Schema Governance

### 11.1 Schema Registry Governance

Schema Registry provides schema management and compatibility controls for supported event formats.

Use of Schema Registry alone does not guarantee that every Kafka record is validated by the broker.

Validation and enforcement must be explicitly designed.

### 11.2 Kafka-Native Validation

Kafka-native producers must use approved event contracts.

Controls may include:

- Producer-side schema validation.
- Schema Registry compatibility rules.
- Broker-side schema validation where supported and configured.
- Topic-level access restrictions.
- Contract testing.
- Downstream validation where necessary.

### 11.3 Platform-Mediated Validation

Platform-mediated sources may supply data in formats such as:

- CSV.
- XML.
- Flat files.
- Database rows.
- JSON.
- Source-specific event envelopes.

These records may require:

- Field mapping.
- Type conversion.
- Schema transformation.
- Data quality validation.
- Enrichment.
- Masking.
- Canonicalisation.
- Lineage metadata.

Transformation may be implemented using managed connectors, Flink, approved ingestion applications or other governed processing components.

### 11.4 Raw and Trusted Topics

Where a source cannot satisfy the trusted event contract at initial ingestion, the solution should separate raw ingestion from trusted publication.

Example:

```text
Source System
      |
      v
Managed Ingestion
      |
      v
Raw Kafka Topic
      |
      v
Validation / Transformation
      |
      +---- Invalid ----> Quarantine
      |
      v
Trusted Kafka Topic
```

Raw topics must not be represented as trusted canonical event streams merely because they are hosted in Kafka.

### 11.5 Trusted Publication Requirement

Before records are made available through trusted MEDA topics, the ingestion solution must establish the applicable:

- Schema conformance.
- Source identity.
- Data quality.
- Authorisation.
- Lineage.
- Data classification.
- Error-handling requirements.

---

## 12. Automated Persistence and State Management

Trusted Kafka streams may be consumed by downstream persistence and processing services.

Supported architectural options include:

### 12.1 Object Storage

Approved S3 sink capabilities may persist curated event streams into MEDA-managed S3 buckets.

Typical uses include:

- Historical retention.
- Replay datasets.
- Analytics.
- Audit storage.
- Data lake integration.

### 12.2 Operational Databases

Approved sink connectors or applications may persist operational state into PostgreSQL, Aurora or other supported data stores.

Typical uses include:

- Materialised views.
- Operational queries.
- Application state.
- Dashboarding.
- Reporting.

### 12.3 Downstream Microservices

MEDA microservices may consume trusted Kafka topics to implement event-driven business capabilities.

### 12.4 Architectural Rule

Persistence targets are selected according to business requirements.

S3 and RDS are not mandatory components of every Pattern 4 implementation.

---

## 13. Delivery Semantics, Idempotency and Replay

### 13.1 Delivery Guarantees

Each ingestion implementation must document its delivery semantics.

At-least-once delivery is a common baseline for mediated ingestion, unless stronger guarantees are explicitly supported and demonstrated end to end.

The solution must not assume exactly-once delivery merely because one component supports transactional processing.

### 13.2 Duplicate Handling

Retries and recovery operations may result in duplicate records.

Where duplicates would affect business correctness, the solution must implement an appropriate strategy such as:

- Stable event identifiers.
- Idempotent consumers.
- Deduplication.
- Transactional processing where supported.
- Source sequence tracking.

### 13.3 Ordering

Ordering guarantees must be documented.

For Kafka, ordering is generally scoped to a partition.

For EventBridge and other mediated ingestion mechanisms, the solution must not assume that source event ordering will be preserved through every integration component.

### 13.4 Replay and Recovery

The ingestion design must define:

- Source replay capabilities.
- Kafka retention requirements.
- Connector offset or checkpoint management.
- File-processing checkpoints.
- CDC positions.
- Recovery after partial batch processing.
- Quarantine replay procedures.
- Prevention of duplicate side effects.

### 13.5 CDC Considerations

CDC implementations must distinguish between:

- Initial database snapshots.
- Ongoing change events.
- Source schema changes.
- Transaction ordering.
- Deletes and tombstones.
- Connector recovery and resynchronisation.

---

## 14. Resiliency and Blast Radius Isolation

### 14.1 Connector Isolation

Each ingestion mechanism should operate within an independently manageable failure boundary.

For example, failures in:

- SFTP ingestion.
- JDBC polling.
- CDC replication.
- EventBridge-to-Kafka publication.

must not unnecessarily affect unrelated ingestion pathways.

### 14.2 Failure Handling

The solution must define:

- Retry policies.
- Retry exhaustion behaviour.
- Dead-letter handling.
- Quarantine.
- Alerting.
- Manual recovery procedures.
- Replay eligibility.

### 14.3 Dead-Letter Governance

Where supported, malformed or non-compliant records are isolated into governed error-handling destinations.

Examples include:

```text
<source-topic>.deadletter
<source-topic>.quarantine
```

The naming convention is illustrative and must align with approved MEDA topic standards.

Dead-letter destinations must be subject to appropriate access controls, retention and monitoring.

### 14.4 Backpressure and Capacity

The ingestion solution must account for:

- Source throughput.
- Batch sizes.
- Connector capacity.
- Kafka quotas.
- Processing capacity.
- Consumer lag.
- Downstream availability.
- Recovery throughput.

Heavy source ingestion workloads must not be allowed to create uncontrolled pressure on shared streaming infrastructure.

---

## 15. Observability, Audit and Lineage

All ingestion mechanisms must provide sufficient operational visibility.

### 15.1 Operational Metrics

Relevant metrics may include:

- Records received.
- Records published.
- Records rejected.
- Connector status.
- Processing latency.
- Kafka producer errors.
- Consumer lag.
- Retry counts.
- Quarantine volume.
- EventBridge delivery failures.
- Source checkpoint status.

### 15.2 Audit Requirements

The ingestion design must provide traceability for:

- Source system.
- Ingestion service.
- Kafka publishing identity.
- Destination topic.
- Processing outcome.
- Relevant timestamps.
- Error conditions.
- Replay operations.

### 15.3 Data Lineage

Where required, lineage must connect the original source system to the resulting trusted Kafka event.

For example:

```text
Source System
     |
     v
Ingestion Service
     |
     v
Raw Kafka Topic
     |
     v
Transformation
     |
     v
Trusted Kafka Topic
     |
     v
Downstream Consumer
```

The lineage model must not lose source identity when an intermediate connector or publishing service is used.

---

## 16. Security and Data Protection

All implementations must comply with applicable TMR and MEDA security requirements.

### 16.1 Authentication and Authorisation

- Use approved authentication mechanisms.
- Apply least-privilege access.
- Separate source and Kafka publishing identities.
- Restrict access to required topics and services.
- Avoid shared credentials across unrelated producers.

### 16.2 Encryption

- Encrypt data in transit.
- Encrypt sensitive data at rest.
- Use approved key-management controls.
- Protect integration credentials and secrets.

### 16.3 Data Classification

The ingestion solution must identify the applicable data classification and associated handling requirements.

These may influence:

- Topic access.
- Data masking.
- Retention.
- Encryption.
- Logging.
- External partner access.
- Storage location.

### 16.4 Secret Management

Credentials must be managed using approved secret-management capabilities.

Credentials must not be embedded in source code, connector definitions or event payloads.

---

## 17. Architecture Implementation Checklist

Before adopting this pattern, the solution architect must confirm the following.

| Area | Required assessment |
|---|---|
| Ingestion lane | Has the appropriate lane been selected? |
| Source capability | Does the source support Kafka, EventBridge, files, CDC or another approved protocol? |
| Source ownership | Is the producing application or system identified? |
| Identity | Are source and Kafka publishing identities defined? |
| Connectivity | Are approved network paths and endpoints documented? |
| Kafka access | Are topic permissions restricted appropriately? |
| Event contract | Is the target schema defined and governed? |
| Validation | Are raw and trusted publication boundaries clear? |
| Transformation | Are mapping, enrichment and masking requirements identified? |
| Provenance | Can the original source be identified downstream? |
| Delivery | Are delivery semantics documented? |
| Duplicates | Is idempotency or duplicate handling addressed? |
| Ordering | Are ordering requirements and limitations understood? |
| Replay | Is recovery and replay behaviour defined? |
| Failure handling | Are retry, DLQ and quarantine arrangements documented? |
| Observability | Are metrics, logs and alerts defined? |
| Capacity | Have throughput, latency and payload sizes been assessed? |
| Data protection | Are classification, retention and encryption requirements satisfied? |
| Operations | Is ongoing ownership and support responsibility assigned? |

---

## 18. The Neutral Protocol Outcome

The Governed Multi-Protocol Integration Layer ensures that differences in source technology do not result in inconsistent governance across the MEDA Event Backbone.

Source systems may communicate using:

- Kafka.
- AWS EventBridge.
- S3.
- SFTP.
- JDBC.
- CDC.
- HTTP.
- Other approved integration protocols.

Each protocol terminates at its appropriate governed integration boundary.

Beyond that boundary, trusted event streams must be:

- Schema-governed.
- Identity-governed.
- Access-controlled.
- Lineage-tracked.
- Operationally observable.
- Recoverable.
- Suitable for event-driven consumption.

The resulting architecture enables TMR to onboard both modern and legacy systems without requiring every producer to implement the Kafka protocol.

**Final architectural principle:**

> Protocol diversity is permitted at the integration boundary. Governance consistency is mandatory at the trusted MEDA Event Backbone.

---

## 19. Pattern Relationships

### Pattern 3 – Identity Firewall

Pattern 4 extends the identity separation principles established in Pattern 3.

Pattern 3 defines the security and identity governance model, while Pattern 4 applies those principles to multiple ingestion protocols and integration mechanisms.

### Related Architectural Capabilities

This pattern may be used alongside architectural guidance for:

- Kafka topic governance.
- Event schema management.
- Event-driven microservices.
- CDC and data replication.
- Event processing with Flink.
- Enterprise API integration.
- Secure external partner integration.
- Data persistence and analytics.

---

## 20. Pattern Identity and Repository Structure

This pattern is maintained as a first-class architectural pattern within the MEDA architecture patterns repository.

**Stable enterprise asset identifier:**

```text
pattern/governed-multi-protocol-integration-layer
```

**GitHub ingestion metadata:**

```yaml
---
id: governed-multi-protocol-integration-layer
---
```

The `id` value is the stable pattern identifier.

The GitHub ingestion service derives the enterprise asset ID by applying the `pattern/` namespace.

The identifier must not depend on the pattern's directory, category or repository location.

**Recommended repository structure:**

```text
meda-architecture-patterns/
└── event-driven/
    └── Governed Multi Layer Protocol/
        ├── README.md
        └── diagrams/
            └── architecture.png
```

The `event-driven` directory provides an organisational classification and does not form part of the enterprise asset identifier.

The pattern's applicability to internal and external producers is defined by its architectural guidance rather than encoded into its asset ID.

Supporting architecture diagrams and documentation are stored in the GitHub repository and published to approved S3 storage through the IDP GitHub ingestion process.

The resulting pattern observation is published to `asset.observations.v1` by Flink Job 2.

Flink Job 6 independently consumes the observation, preserves the pattern's standalone enterprise identity and publishes the governed asset representation to `asset.state.v1` for consumption by Backstage.
