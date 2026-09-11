---
title: "Designing a Scalable Mobile Backend: BFF, API Gateway, Identity and Sessions"
description: "A practical architecture guide for mobile backends that must remain secure, observable and compatible as products and teams grow."
categories: [Software Architecture, Mobile]
tags: [mobile-backend, bff, api-gateway, keycloak, oidc, security, microservices]
---

A mobile application is never only a mobile application. Behind every screen there is an identity flow, a set of APIs, version compatibility rules, operational dependencies and a security boundary. When these concerns are handled independently by each backend service, the mobile team inherits a fragmented platform. Delivery slows down, incidents become harder to diagnose and seemingly small product changes begin to require coordinated releases across many teams.

The goal of a mobile backend architecture is therefore not merely to expose data. It must provide a stable contract between a fast-moving client and a distributed enterprise platform.

## Start with the client contract

Mobile clients and web clients evolve differently. A web frontend can usually be deployed together with its backend. A mobile release, however, remains in app stores and on user devices for months. The backend must continue supporting older versions while newer clients introduce additional fields, flows and security requirements.

This makes contract design a first-class architectural concern. A useful baseline includes:

- explicit API versioning and a documented deprecation policy;
- backward-compatible response changes wherever possible;
- capability negotiation for features that depend on client version;
- idempotency for payment, enrollment and other critical commands;
- consistent error codes that the client can map to user actions;
- correlation identifiers carried from the device to downstream services.

The contract should reflect the mobile journey rather than exposing the internal service topology. A customer opening a policy, payment or profile screen should not need the client to coordinate five internal services.

## Where BFF fits

A Backend for Frontend is valuable when the mobile experience needs orchestration, aggregation or device-specific behavior. It can combine several internal responses, shape payloads for constrained networks and hide internal service changes from the application.

A BFF should not become a second business layer. Core pricing, eligibility, payment and authorization rules still belong in domain services. The BFF owns presentation-oriented orchestration:

- aggregating data required by one screen;
- translating internal errors into a stable client contract;
- applying mobile-specific response shaping;
- coordinating feature flags and minimum-version rules;
- reducing unnecessary network round trips.

The boundary is important. If business decisions accumulate in the BFF, the same rules are soon reimplemented for web, partner and branch channels.

## API Gateway and BFF solve different problems

These two components are often treated as alternatives, but they operate at different levels.

The API Gateway protects and manages the platform edge. It handles routing, TLS termination, rate limits, request-size controls, coarse-grained access policies and operational telemetry. The BFF sits behind that edge and adapts platform capabilities to a particular client experience.

A common request path is:

Mobile client → API Gateway → Mobile BFF → Domain services

Keeping these responsibilities separate makes governance clearer. Security teams can define edge policies centrally, while product teams can evolve mobile orchestration without changing the platform gateway for every screen.

## Identity is an end-to-end flow

Using an identity provider does not by itself create a secure mobile architecture. The complete flow must be designed around public-client constraints.

For native applications, Authorization Code Flow with PKCE is the normal foundation. Credentials should never be embedded in the application. Tokens must be stored using the operating system's secure storage, refreshed intentionally and invalidated when device or account risk changes.

The backend must answer several operational questions:

- Which claims are trusted, and which service verifies them?
- How are roles and fine-grained permissions separated?
- What happens when an employee or customer loses access?
- How are concurrent sessions listed and revoked?
- How are refresh-token reuse and suspicious device changes detected?
- Which downstream calls propagate user identity, and which use service identity?

Keycloak or another standards-based provider can support these flows, but architecture decisions still belong to the application and security teams. Authentication proves identity; authorization decides whether a specific action is allowed.

## Session management without hidden state

A fully stateless access-token model is attractive, but most enterprise mobile products eventually need controlled session state. Device registration, risk signals, refresh-token rotation, notification preferences and forced logout all require lifecycle management.

The practical approach is to keep access tokens short-lived and place only stable claims in them. Mutable information should be evaluated by the appropriate service or policy layer. Session records can link the account, device installation and refresh-token family without turning every API call into a database lookup.

Logout must be defined precisely. Local token deletion, server-side refresh revocation and global account logout are different operations. The product must decide which one each user action represents.

## Push notifications are part of the backend

Push delivery is frequently implemented as a utility added near release. In reality, it is a distributed workflow with privacy, retry and consistency requirements.

The backend should store device installations separately from user accounts because a user may have several devices and a device token may change. Business services should publish notification intents rather than call Apple or Google providers directly. A notification service can then apply templates, preferences, quiet hours, retries and delivery telemetry.

Sensitive information should not be placed in a notification payload. The notification can invite the application to fetch authorized data after the user opens it.

## Design for failure and observation

A mobile request may cross the gateway, BFF, identity service, cache, message broker and several domain services. Without common telemetry, the user reports “the screen is loading” while every team sees healthy individual services.

Use a correlation identifier across the complete path. Track latency and errors by endpoint, client version and dependency. Distributed tracing is especially useful for aggregated BFF calls, but it should be supported by business-level metrics such as successful login, payment completion and notification processing time.

Timeouts and retries must be owned deliberately. A retry at the mobile client, gateway, BFF and service can multiply one request into an incident. Retry only idempotent operations, add jitter and impose a clear time budget for the complete user journey.

## A decision framework

Before adding another component, evaluate the decision against five questions:

1. Does it stabilize the mobile contract?
2. Does it reduce coupling to internal services?
3. Can security policy be enforced and audited?
4. Can an incident be traced from the device to the responsible dependency?
5. Can the platform support old and new client versions during a controlled transition?

A scalable mobile backend is not defined by the number of microservices. It is defined by clear boundaries, controlled change and the ability to operate the whole customer journey. BFF, API Gateway, identity and session management are useful patterns only when each has an explicit responsibility and measurable operational behavior.
