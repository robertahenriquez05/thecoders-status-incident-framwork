# THECODERS | ARCHITECTURE DECISION RECORD

## ADR-001 Keep Subscriber Access Accountless

**Authentication boundary for the Service Status and Incident Framework**

**Team:** TheCoders — Service Status and Incident Framework

**Course:** Software Engineering

**Date:** September 28, 2026

**Decision makers:** TheCoders team - Roberta Henriquez, Sam Morris, Rommel Mensah, Justin Brown

**Related requirements:** T2 Requirements Specification, Gap #1; FR-13-FR-16 and FR-20-FR-22

## Context

The service status and incident framework has two audiences with different needs. Team members need authenticated access to manage components and incidents. Subscribers need to sign up for updates filtered by component or severity and choose email, webhook, or both. They also need to unsubscribe without creating an account (FR-16). The charter limits v1 to two roles, but did not say whether a subscriber was an account or simply a contact with a subscription. Leaving that unresolved would affect the data model, signup and unsubscribe flows, and access-control work.

The team reviewed this gap while producing the requirements specification. On September 21, 2026, all four members confirmed the working assumption that subscribers are unauthenticated and only admins log in (T2, Section 6, Gap #1). The decision fits the anonymous signup and token-based unsubscribe requirements already described in the charter and user stories.

## Decision

For v1, subscribers will not have login accounts or authenticated sessions. A subscription is identified by its email address and/or webhook destination, its filters, and a private unsubscribe token. Admins are the only users who authenticate; admin management views require authentication. Unsubscribe links use the subscriber's token and do not require login.

## Alternatives Considered

### 1. Give subscribers accounts and require login to manage subscriptions.

This would give subscribers a familiar authenticated place to manage preferences and could support multiple subscriptions under one identity. We did not choose it because it adds account creation, password recovery, session handling, and credential support to a v1 whose specified subscriber flow is signup by contact details and token-based unsubscribe. It also conflicts with the explicit requirement that unsubscribing not require an authenticated account.

### 2. Allow accountless signup, but require email verification or a one-time login link before managing preferences.

This would provide evidence of control over an email address and reduce the risk of someone changing another person's subscription. We did not choose it because it adds a verification step and delivery/failure cases to signup, while the current requirements only call for an unsubscribe token and do not require a subscriber portal or verified address. Webhook subscribers also do not necessarily have an email address to verify.

### 3. Use one shared authentication model for both admins and subscribers, with separate permissions.

This would provide one identity and authorization system. We did not choose it because subscribers need notification preferences, not access to admin capabilities, and the charter fixes the role set and public/admin view split for v1. Combining the models would add permission and account complexity without a stated subscriber need.

These alternatives capture the competing concerns made visible by the gap: stronger identity and self-service controls versus a low-friction, accountless subscription flow. The requirements record the team's selected resolution, but do not attribute a particular alternative or argument to an individual member.

## Consequences

### Benefits

- Subscribers can sign up and unsubscribe without creating or remembering a password.
- The authentication boundary stays small: admin views require login, while the public status page and subscriber signup remain public.
- The data model can represent subscriptions and their filters without requiring every subscriber to be an application user.

### Costs and constraints

- Unsubscribe tokens act as bearer credentials: anyone with a valid link may unsubscribe that subscription. Tokens must be unpredictable, stored and handled carefully, and invalidated when used.
- Without subscriber accounts, a subscriber cannot use an authenticated portal to review or change preferences; changing them may require a new signup flow.
- The system cannot establish that an email address is controlled by the person who entered it unless a separate verification step is added later.
- Accountless subscriptions need careful duplicate handling and token lifecycle rules, and webhook destinations still need input validation and safe delivery behavior.
- If future requirements demand subscriber portals, verified identities, or shared preferences across destinations, the team may need to migrate subscription records or add a separate subscriber identity model.

## References

T2 Requirements Specification, Sections 2.4, 2.6, 6 (Gap #1), and 8.

Team Charter / Project Brief v0.1, as referenced by the T2 specification.
