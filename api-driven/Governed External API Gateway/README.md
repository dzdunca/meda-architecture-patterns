---
id: governed-external-api-gateway
---

# Governed External API Gateway

## 1. Introduction

The **Governed External API Gateway** architectural pattern defines how external consumers securely access approved TMR APIs through a centrally governed, public-facing API entry point into the private TMR MEDA platform.

The architecture uses **Amazon API Gateway** as the external entry point and **MuleSoft Flex Gateway**, deployed on Amazon Elastic Kubernetes Service (EKS), as the internal API policy enforcement and routing capability.

The pattern establishes a layered security architecture that supports:

- Secure external API exposure through public HTTPS endpoints.
- Controlled access for approved external applications, government partners and third parties.
- External authentication and authorisation.
- AWS Web Application Firewall (WAF) protection where supported and configured.
- API throttling and traffic management.
- Private integration between AWS API Gateway and the MEDA platform.
- Centralised API policy enforcement through MuleSoft Flex Gateway.
- JWT validation using approved identity providers.
- Controlled routing to approved backend services.
- Enterprise logging, monitoring and auditability.
- Separation between external consumer identities, gateway identities and backend service identities.
- Prevention of unauthorised direct access to internal services.

Backend destinations may include:

- Microservices deployed on Amazon EKS.
- Internal TMR enterprise applications.
- Approved SaaS API endpoints accessed through governed connectivity.

**Architectural principle:** External consumers access TMR APIs through a controlled public entry point, while internal API governance, policy enforcement and backend routing remain within the private MEDA platform.

---

## 2. Purpose and Architectural Outcomes

TMR requires a consistent and secure approach for exposing approved business APIs to consumers outside its internal network.

External consumers may include:

- Government agencies.
- Approved business partners.
- Third-party suppliers.
- External applications.
- Authorised customer systems.

Allowing individual backend applications to expose their own public endpoints would introduce inconsistent security controls, duplicated API management capabilities and unnecessary exposure of internal infrastructure.

The Governed External API Gateway establishes a standard approach that separates external API access from internal service implementation.

### 2.1 Architectural Outcomes

The pattern provides:

- A single governed architectural approach for external API exposure.
- Centralised management of public API entry points.
- Layered authentication, authorisation and threat protection.
- Private connectivity into the MEDA platform.
- Consistent API policy enforcement.
- Controlled backend routing.
- Separation of external and internal trust boundaries.
- Standardised logging, monitoring and auditability.
- Independent management of API gateways and backend applications.
- Reduced risk of exposing internal services directly to the internet.

### 2.2 Core Architectural Decisions

| Decision | Architectural requirement |
|---|---|
| External API entry point | AWS API Gateway provides the approved public HTTPS entry point. |
| API management | MuleSoft Flex Gateway provides internal API policy enforcement and backend routing. |
| Runtime | Flex Gateway is deployed within the approved MEDA Amazon EKS environment. |
| Private connectivity | AWS API Gateway connects through VPC Link to an approved internal load balancer. |
| External protection | Authentication, throttling and supported threat-protection controls are applied at the external boundary. |
| Internal policy enforcement | Flex Gateway enforces approved API policies, including JWT validation where required. |
| Backend access | Backend services remain accessible only through approved network paths. |
| Identity | External, gateway and backend identities are governed independently. |
| API contracts | Externally exposed APIs maintain approved and versioned API specifications. |
| Observability | Gateway and backend telemetry integrate with approved enterprise monitoring. |
| Configuration | API and gateway configuration is version-controlled and deployed through approved processes. |
| Bypass prevention | External consumers cannot directly access Flex Gateway or internal backend services. |

---

## 3. When to Use This Pattern

Use this pattern when an approved API must be accessible to consumers outside the internal TMR network.

Typical scenarios include:

- Government-to-government API integration.
- External partner access to TMR business capabilities.
- Approved third-party applications consuming TMR APIs.
- External applications requiring secure access to MEDA-hosted services.
- Public HTTPS endpoints requiring centralised security controls.
- APIs requiring external authentication and traffic governance.
- APIs whose backend services must remain privately accessible.

### 3.1 When Not to Use This Pattern

This pattern should not be used when:

- The API is consumed exclusively by internal TMR applications. Use the **Governed Internal API Gateway** pattern.
- The integration is asynchronous and event-driven rather than an API request-response interaction.
- A backend service does not require external exposure.
- The proposed architecture would bypass required MEDA API management policies.
- An external consumer requires direct access to internal Kubernetes services or private infrastructure.

### 3.2 Selection Guidance

| Requirement | Recommended approach |
|---|---|
| External HTTPS API exposure | Governed External API Gateway |
| Internal TMR API exposure | Governed Internal API Gateway |
| External threat protection | AWS API Gateway with applicable WAF and security controls |
| Private connectivity into MEDA | API Gateway VPC Link and internal load balancer |
| API policy enforcement | MuleSoft Flex Gateway |
| JWT validation | Approved Flex Gateway JWT policy |
| API throttling | Approved gateway throttling and rate-limiting controls |
| EKS microservice backend | Approved Kubernetes Service |
| Internal enterprise application backend | Approved private enterprise endpoint |
| SaaS API backend | Approved SaaS endpoint with governed connectivity |
| Asynchronous event ingestion | Governed Multi-Protocol Integration Layer |

---

## 4. Solution Architecture

### 4.1 Solution Architecture Diagram

![Governed External API Gateway – Solution Architecture](diagrams/governed_external_api_gateway_V1.png)

The architecture separates the public-facing API entry point from the private MEDA API management and backend environment.

### 4.2 Logical Architecture

![Governed Multi-Protocol Integration Layer – Inbound Architecture](diagrams/governed_external_inbound_V1.png)

The policy controls shown are logical responsibilities and do not represent separate network hops.

### 4.3 External API Boundary

AWS API Gateway provides the public HTTPS entry point for approved external consumers.

Its responsibilities include:

- Receiving external HTTPS requests.
- Enforcing configured authentication and authorisation.
- Applying API throttling and quotas where configured.
- Integrating with applicable AWS WAF protections.
- Routing requests through the approved private integration.
- Capturing external API access logs and operational metrics.

### 4.4 Private Integration Boundary

AWS API Gateway uses VPC Link to access an approved internal load balancer.

The private integration boundary ensures that external API requests reach the MEDA platform without requiring a public endpoint on Flex Gateway.

### 4.5 Internal API Management Boundary

MuleSoft Flex Gateway provides internal API policy enforcement and routing.

Its responsibilities include:

- JWT validation.
- Approved authorisation policy enforcement.
- Rate limiting.
- Request and response logging.
- API route management.
- Backend destination selection.
- Operational telemetry.

### 4.6 Backend Services

Backend destinations are independently deployed and governed.

The pattern does not require the backend to be an EKS microservice.

Approved destinations may include:

- Kubernetes-hosted microservices.
- Internal enterprise application endpoints.
- SaaS APIs accessible through approved TMR network and security controls.

The selected destination must comply with applicable authentication, authorisation, encryption and connectivity requirements.

---

## 5. Architectural Governance and Tenancy

### 5.1 External API Plane – AWS

The external API plane operates within the approved TMR AWS environment.

It includes:

- AWS API Gateway.
- Public API custom domains where required.
- TLS certificates.
- API routes and integrations.
- VPC Link.
- Applicable WAF protections.
- CloudWatch logs and metrics.
- IAM roles and permissions.

The external API plane is governed through approved platform security and deployment processes.

### 5.2 Internal API Management Plane – MEDA

The internal API management plane uses MuleSoft Flex Gateway deployed on Amazon EKS.

Its components may include:

- Flex Gateway runtime.
- API instance definitions.
- API route configurations.
- JWT validation policies.
- Rate limiting policies.
- Request and response logging policies.
- Backend destination definitions.
- Gateway observability components.

### 5.3 Backend Application Plane

Backend applications are managed independently of the external API entry point.

Responsibilities include:

- Business logic.
- Application-level authorisation.
- Data access.
- Backend authentication.
- Service availability.
- Application monitoring.
- Application deployment and lifecycle.

### 5.4 Governance Responsibilities

| Responsibility | Primary capability |
|---|---|
| Public API exposure | AWS API Gateway |
| Perimeter security | AWS API Gateway and applicable WAF controls |
| Private network integration | VPC Link and internal load balancer |
| Internal API management | MuleSoft Flex Gateway |
| API security policies | Flex Gateway and approved identity services |
| Backend business processing | Backend application |
| Operational monitoring | Approved AWS, MuleSoft and enterprise monitoring services |
| API contracts | API-owning team under MEDA governance |
| Deployment governance | Approved CI/CD and infrastructure-as-code processes |

---

## 6. External API Gateway Configuration Model

The AWS API Gateway configuration must define the external API contract and its approved private integration.

| Component | Responsibility |
|---|---|
| AWS API Gateway | Public HTTPS API endpoint |
| API routes | Maps external methods and paths to approved integrations |
| Authoriser | Enforces configured external authentication and authorisation |
| AWS WAF | Provides supported web application threat protection |
| Throttling | Controls external request volume |
| VPC Link | Provides private integration connectivity |
| Internal load balancer | Routes approved traffic to the Flex Gateway runtime |
| MuleSoft Flex Gateway | Enforces MEDA API policies |
| Backend destination | Provides the requested business capability |

### 6.1 API Gateway Type

The approved API Gateway type must be selected based on required capabilities.

For example, REST APIs and HTTP APIs have different support for:

- Authorisation mechanisms.
- WAF association.
- Usage plans and API keys.
- Private integration options.
- Request validation.
- Transformation capabilities.

The implementation must not assume all API Gateway features are available for every API type.

### 6.2 API Route Configuration

Each externally exposed route must define:

- HTTP method.
- Public API path.
- Authentication requirements.
- Target private integration.
- Backend route mapping.
- Request and response behaviour.
- Applicable throttling.
- Logging requirements.

### 6.3 Environment Separation

External API configuration must support approved environment boundaries.

Environment-specific values include:

- API domain names.
- Gateway identifiers.
- VPC Link configuration.
- Internal load balancer destinations.
- Identity provider settings.
- Logging configuration.
- Security policies.

---

## 7. Private Connectivity – External to Internal Boundary

### 7.1 Connectivity Model

The approved connectivity path is:

```text
External Consumer
       |
       v
AWS API Gateway
       |
       v
VPC Link
       |
       v
Internal Load Balancer
       |
       v
MuleSoft Flex Gateway
       |
       v
Approved Backend
```

### 7.2 VPC Link

VPC Link provides the private integration between AWS API Gateway and the approved internal load balancer.

The configuration must define:

- Supported API Gateway integration type.
- VPC Link ownership.
- Network routing.
- Security groups.
- Target load balancer.
- Destination ports.
- Health and availability requirements.

### 7.3 Internal Load Balancer

The internal load balancer provides controlled routing to the Flex Gateway runtime.

Requirements include:

- Internal-only exposure.
- Approved listener configuration.
- Appropriate target groups.
- Health checks.
- Restricted network access.
- Encryption in transit according to security requirements.

### 7.4 Backend Network Isolation

Backend services must not become publicly accessible merely because an API is exposed through AWS API Gateway.

Network policies, routing and application access controls must prevent unauthorised alternative paths.

### 7.5 Connectivity to SaaS Backends

Where Flex Gateway routes to an approved SaaS API, the architecture must define:

- Approved outbound connectivity.
- Destination hostname and DNS resolution.
- TLS validation.
- Authentication and credential handling.
- Proxy or egress controls where required.
- Data classification and residency requirements.

SaaS destinations do not need to run inside EKS or the TMR network, but access to them must be governed.

---

## 8. External Authentication and Authorisation

### 8.1 External Consumer Identity

External consumers must authenticate through approved identity mechanisms appropriate to the API.

Supported approaches depend on the API Gateway type and configured authorisation capability.

Examples may include:

- OAuth 2.0 access tokens.
- JWT authorisation.
- Lambda authorisers.
- Other approved authentication mechanisms.

### 8.2 Identity Provider Integration

The architecture must define:

- Approved identity provider.
- Token issuer.
- Token audience.
- Client or application registration.
- Required scopes or claims.
- Token lifetime.
- Credential rotation.
- Revocation or access removal procedures.

### 8.3 Authentication Enforcement

External authentication must be enforced at the approved boundary.

A request must not reach an internal backend if it fails the applicable authentication requirements.

### 8.4 Authorisation

Authentication confirms the caller's identity or credentials.

Authorisation determines whether that caller is permitted to invoke a particular API operation.

Authorisation may be enforced at multiple layers:

- AWS API Gateway.
- MuleSoft Flex Gateway.
- Backend application.

The responsibilities of each layer must be explicit.

### 8.5 API Keys

Where API keys are used, they must not be treated as a substitute for strong authentication unless explicitly approved for the use case.

API keys may support consumer identification, usage management or quota enforcement depending on the selected API Gateway capability.

---

## 9. Perimeter Security and Threat Protection

### 9.1 AWS WAF

Where supported by the selected API Gateway type, AWS WAF may provide additional protection against common web application threats.

Applicable controls may include:

- Managed rule groups.
- IP-based restrictions.
- Request filtering.
- Rate-based rules.
- Approved threat signatures.

For API Gateway configurations that do not support direct WAF association, an alternative approved protection architecture must be assessed.

### 9.2 API Throttling

AWS API Gateway must apply appropriate request throttling based on:

- Expected consumer traffic.
- Backend capacity.
- Peak usage.
- Abuse prevention requirements.
- Service availability objectives.

### 9.3 Request Validation

The external gateway should enforce supported request validation controls where appropriate.

Validation may include:

- Required parameters.
- Request format.
- Content type.
- Payload size.
- Supported HTTP methods.

Full business validation remains the responsibility of the appropriate internal policy or backend application.

### 9.4 Transport Security

Public API endpoints must use approved TLS configurations.

Certificates must be issued, managed and renewed through approved processes.

### 9.5 Protection Against Gateway Bypass

External consumers must not have direct public access to:

- Flex Gateway listeners.
- Internal load balancers.
- EKS application services.
- Internal enterprise application endpoints.

Approved SaaS providers may maintain their own public endpoints; this pattern governs the TMR API access path rather than asserting control over the SaaS provider's own exposure.

---

## 10. Internal API Policy Enforcement – MuleSoft Flex Gateway

### 10.1 Purpose

MuleSoft Flex Gateway provides consistent internal API management after the request crosses the private integration boundary.

### 10.2 JWT Validation

Flex Gateway may validate:

- Token signature.
- Approved issuer.
- Audience.
- Token expiration.
- Not-before constraints.
- Required claims.
- Signing-key information.

The applicable controls must be configured and validated against the approved identity provider.

### 10.3 Authorisation Policies

Flex Gateway may enforce additional API access policies where supported.

These policies must complement rather than unintentionally conflict with external gateway authorisation.

### 10.4 Rate Limiting

Flex Gateway may apply internal rate limits independently of AWS API Gateway throttling.

For example:

- AWS API Gateway protects the public entry point.
- Flex Gateway protects the backend service or enforces API-specific consumer limits.

These controls should be coordinated to avoid unnecessary duplication or inconsistent responses.

### 10.5 API Routing

Flex Gateway routes approved requests to configured backend destinations.

Routing may be based on:

- API instance.
- Request path.
- HTTP method.
- Approved backend service definition.
- API version.

### 10.6 Policy Governance

Gateway policies must be:

- Version-controlled.
- Reviewed.
- Approved.
- Tested.
- Deployed through approved processes.
- Monitored.
- Auditable.

---

## 11. Identity Firewall and Trust Boundary

### 11.1 Purpose

The architecture establishes a separation between external consumer identity and internal service identity.

The external consumer must not automatically inherit privileges within the MEDA platform.

### 11.2 Identity Boundaries

```text
External Consumer Identity
            |
            v
AWS API Gateway
(External Authentication)
            |
            v
Private Integration Boundary
            |
            v
MuleSoft Flex Gateway
(Internal Policy Enforcement)
            |
            v
Backend Service Identity
```

### 11.3 Identity Propagation

Where downstream services require caller context, the architecture must define which identity claims or metadata may be propagated.

Caller context must not be accepted from untrusted headers without appropriate validation.

### 11.4 Service-to-Service Authentication

The gateway-to-backend connection must use the authentication mechanism required by the destination.

For example:

- Internal service authentication.
- Approved OAuth client credentials.
- Managed service credentials.
- Other supported enterprise authentication mechanisms.

### 11.5 Least Privilege

Each identity must have only the permissions required for its role.

This applies to:

- External consumers.
- API Gateway integration roles.
- Gateway runtime identities.
- Backend service credentials.
- Operational administrators.

### 11.6 Trust Boundary Enforcement

Crossing from the public API endpoint into private infrastructure does not automatically establish application-level trust.

Authentication, authorisation and policy enforcement must continue at the appropriate layers.

---

## 12. Backend Routing and Service Integration

### 12.1 EKS Microservices

For Kubernetes-hosted backends, Flex Gateway may route requests to an approved Kubernetes Service.

Example:

```text
Flex Gateway
      |
      v
Kubernetes Service
      |
      v
Backend Microservice
```

The implementation must define:

- Namespace.
- Service name.
- Port.
- Network policy.
- Health checks.
- Application authentication.

### 12.2 Internal Enterprise Applications

Flex Gateway may route to approved internal enterprise application endpoints.

The architecture must define:

- Private network connectivity.
- DNS resolution.
- TLS requirements.
- Authentication.
- Availability.
- Timeout and retry behaviour.

### 12.3 Approved SaaS APIs

Flex Gateway may also route to an approved SaaS API.

The integration must address:

- Outbound network connectivity.
- API authentication.
- Credential management.
- TLS.
- Vendor availability.
- Request and response handling.
- Data protection obligations.

### 12.4 Backend Independence

The external API architecture must not assume a particular backend technology.

The gateway is responsible for governed API access and routing.

The backend remains responsible for its business capability and application-level controls.

---

## 13. API Contract Governance and Lifecycle

### 13.1 OpenAPI Specification

Externally exposed APIs should maintain an approved OpenAPI specification where applicable.

The specification must describe:

- API resources.
- Supported methods.
- Request parameters.
- Request and response schemas.
- Authentication requirements.
- Error responses.
- API version.
- Consumer expectations.

### 13.2 API Ownership

Every externally exposed API must have a defined owner responsible for:

- API contract.
- Consumer requirements.
- Security classification.
- Versioning.
- Availability requirements.
- Lifecycle management.

### 13.3 API Versioning

API versions must follow approved MEDA standards.

Version changes must consider:

- Backward compatibility.
- External consumer impact.
- Gateway routing.
- Backend changes.
- Deprecation.
- Retirement.

### 13.4 Consumer Onboarding

External consumers must be onboarded through an approved process.

The process must define:

- Consumer identity.
- Business purpose.
- Approved APIs.
- Access permissions.
- Credentials.
- Usage limits.
- Support arrangements.
- Access revocation.

### 13.5 API Retirement

Retirement must include controlled removal of:

- Public API routes.
- Consumer access.
- Gateway policy configuration.
- Backend routing.
- Unused credentials.
- Related operational configuration.

---

## 14. Logging, Monitoring and Observability

### 14.1 External Gateway Logging

AWS API Gateway provides external API access telemetry through approved AWS monitoring capabilities.

Relevant information may include:

- Request timestamp.
- API identifier.
- HTTP method.
- Request path.
- Response status.
- Request latency.
- Integration latency.
- Authorisation failures.
- Throttling events.

### 14.2 Flex Gateway Logging

Flex Gateway provides internal API request and response telemetry.

Logging may include:

- API instance.
- API version.
- Correlation identifier.
- Backend destination.
- Response status.
- Policy enforcement outcome.
- Processing duration.

### 14.3 Enterprise Observability

Gateway telemetry must integrate with approved enterprise observability capabilities.

For MEDA, this may include:

- Amazon CloudWatch.
- Splunk.
- Approved application monitoring.
- Operational dashboards.
- Alerting.

### 14.4 Correlation and Traceability

The architecture must support tracing an external API request across:

```text
External Consumer
        |
        v
AWS API Gateway
        |
        v
Internal Load Balancer
        |
        v
MuleSoft Flex Gateway
        |
        v
Backend Application
```

An approved correlation identifier should be preserved or propagated across the request path.

### 14.5 Sensitive Data Protection

Operational logs must not unnecessarily contain:

- Access tokens.
- Client secrets.
- API credentials.
- Sensitive request bodies.
- Sensitive response bodies.
- Personal information beyond approved logging requirements.

---

## 15. Resiliency and Availability

### 15.1 External Gateway Availability

AWS API Gateway configuration must meet the availability requirements of the exposed APIs.

### 15.2 Private Integration Availability

The architecture must assess:

- VPC Link availability.
- Internal load balancer health.
- Flex Gateway availability.
- Backend health.
- Network dependencies.

### 15.3 Flex Gateway Resilience

Flex Gateway deployments must meet approved runtime availability requirements.

Considerations include:

- Multiple replicas where supported.
- Appropriate EKS placement.
- Resource requests and limits.
- Readiness and health checks.
- Controlled deployments.
- Recovery procedures.

### 15.4 Backend Failure Isolation

Failure of one backend application should not unnecessarily affect unrelated APIs.

### 15.5 Timeouts and Retries

The implementation must define:

- Connection timeouts.
- Request timeouts.
- Retry behaviour.
- Gateway timeout responses.
- Backend failure responses.
- Client retry guidance.

Retries must be assessed carefully for non-idempotent business operations.

### 15.6 Capacity Planning

Capacity assessments must consider:

- External request volume.
- Peak traffic.
- Gateway throttling.
- Flex Gateway throughput.
- Load balancer capacity.
- Backend capacity.
- SaaS provider rate limits where applicable.

---

## 16. Security and Data Protection

### 16.1 Public Exposure

Only approved AWS API Gateway endpoints may be publicly exposed under this pattern.

### 16.2 Private Integration

Connectivity into MEDA must use approved private integration mechanisms.

### 16.3 Authentication and Authorisation

External authentication and internal API policy enforcement must be configured according to the approved security design.

### 16.4 Encryption

The architecture must protect data in transit using approved TLS configurations.

Encryption requirements apply to:

- External consumer to AWS API Gateway.
- API Gateway private integration.
- Flex Gateway to backend destinations.

The actual encryption boundaries and TLS termination points must be documented.

### 16.5 Secrets Management

Credentials must be stored through approved secret-management capabilities.

Secrets must not be embedded in:

- API specifications.
- Kubernetes manifests.
- Application source code.
- Gateway policy definitions.
- Operational logs.

### 16.6 Data Classification

The API owner must identify applicable data classification and handling requirements.

These may influence:

- External consumer eligibility.
- Logging.
- Retention.
- Encryption.
- Authentication strength.
- Authorisation.
- SaaS integration.
- Data residency.

### 16.7 Security Testing

Externally exposed APIs must undergo applicable security validation before production release.

This may include:

- Authentication testing.
- Authorisation testing.
- API security testing.
- Threat protection validation.
- Gateway bypass testing.
- Configuration review.
- Penetration testing where required.

---

## 17. Architecture Implementation Checklist

Before adopting this pattern, the solution architect must confirm the following.

| Area | Required assessment |
|---|---|
| External exposure | Is public API access approved and required? |
| Consumer ownership | Are external consumers and their business purposes identified? |
| API Gateway | Is the appropriate AWS API Gateway type selected? |
| Public endpoint | Are domain, certificate and HTTPS requirements defined? |
| External authentication | Is the approved identity mechanism configured? |
| Authorisation | Are external consumer permissions defined? |
| WAF | Is the applicable threat-protection approach established? |
| Throttling | Are request limits aligned with API requirements? |
| VPC Link | Is the approved private integration configured? |
| Internal load balancer | Are listener, routing and health requirements defined? |
| Flex Gateway | Is the approved EKS gateway runtime identified? |
| JWT validation | Are issuer, audience, signature and expiry controls defined? |
| Policy governance | Are required API policies configured and tested? |
| Backend destination | Is the backend endpoint approved and reachable? |
| Backend authentication | Are backend identity and access requirements defined? |
| SaaS integration | Are external SaaS connectivity and data handling requirements approved where applicable? |
| Bypass prevention | Are alternative unmanaged access paths prevented? |
| API contract | Is the OpenAPI specification defined and governed? |
| Versioning | Are compatibility and lifecycle requirements documented? |
| Logging | Are gateway and backend logs integrated with approved monitoring? |
| Correlation | Can requests be traced across the gateway layers? |
| Sensitive data | Are tokens, credentials and sensitive payloads protected? |
| Resilience | Are failure, timeout and recovery behaviours defined? |
| Capacity | Have throughput and backend capacity requirements been assessed? |
| Deployment | Are gateway configurations version-controlled and automated? |
| Operations | Are platform and application support responsibilities assigned? |

---

## 18. The Governed External API Outcome

The Governed External API Gateway establishes a consistent, secure and reusable architecture for exposing approved TMR business capabilities to external consumers.

The architecture ensures:

- Controlled public HTTPS API exposure.
- Centralised perimeter security.
- Private connectivity into the MEDA platform.
- Consistent internal API policy enforcement.
- Independent governance of external and internal identities.
- Approved backend routing.
- Protection against unmanaged direct backend access.
- Enterprise logging and observability.
- Repeatable deployment and lifecycle management.
- Clear ownership and operational support.

The pattern allows TMR to expose business capabilities without requiring each application team to independently design and operate its own public API gateway infrastructure.

**Final architectural principle:**

> AWS API Gateway governs external access. MuleSoft Flex Gateway governs internal API policy enforcement and routing. Backend services remain protected behind approved integration boundaries.

---

## 19. Related Architectural Patterns

### Governed Internal API Gateway

The **Governed Internal API Gateway** defines how internal TMR applications consume APIs through MuleSoft Flex Gateway.

It provides:

- Internal API exposure.
- JWT validation.
- API policy enforcement.
- Rate limiting.
- Logging.
- Backend routing.

The Governed External API Gateway extends this architecture by introducing a controlled public entry point and private integration boundary.

The two patterns share the Flex Gateway API management capability but address different consumer trust boundaries.

### Governed Multi-Protocol Integration Layer

The **Governed Multi-Protocol Integration Layer** defines how Kafka-native applications and systems using alternative protocols securely publish events into the MEDA Event Backbone.

It may complement an external API integration that results in asynchronous event publication.

### Governed Private Event Processing

The **Governed Private Event Processing** pattern defines how events are securely processed, enriched, distributed and persisted within the trusted MEDA environment.

It may complement backend services that produce or consume Kafka events.

### Related Architectural Capabilities

This pattern may also be used alongside architectural guidance for:

- Amazon EKS microservice deployment.
- Enterprise identity and access management.
- API contract governance.
- AWS networking and private integration.
- API security and threat protection.
- CI/CD deployment.
- Enterprise observability.
- SaaS integration governance.

---

## 20. Pattern Classification

**Architectural pattern:** Governed External API Gateway

**Category:** API and Integration Architecture

**Enterprise asset identifier:**

`pattern/governed-external-api-gateway`

**Applicability:** Secure external access to approved TMR APIs through AWS API Gateway and the private MEDA API management platform.

**Primary capabilities:**

- Public HTTPS API exposure.
- AWS API Gateway.
- Private integration using VPC Link.
- Internal load balancer routing.
- MuleSoft Flex Gateway on Amazon EKS.
- JWT validation.
- API policy enforcement.
- Throttling and threat protection.
- Backend routing.
- Enterprise observability.
- External identity governance.
- API lifecycle management.

The pattern's stable identifier is defined by the YAML front matter and does not depend on the repository directory or category.

The architecture is reusable across TMR solutions and does not prescribe specific application names, API endpoints, backend technologies or external consumer identities.