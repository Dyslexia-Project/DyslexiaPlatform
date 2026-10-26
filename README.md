# Dyslexia Reader — Platform

The web platform for Dyslexia Reader: a dashboard, account services, synchronized reading preferences, and cloud text-to-speech.

**Repository:** [DyslexiaPlatform](https://github.com/Dyslexia-Project/DyslexiaPlatform). The native desktop client is maintained in [DyslexiaApp](https://github.com/Dyslexia-Project/DyslexiaApp).

**Status:** university project in planning. Features, service plans, and team assignments below are proposals, not shipped capabilities or proven business results.

## Desktop integration

The native C++ desktop client is maintained in a separate repository. It handles screen capture, local OCR, the reading window, appearance settings, and local speech.

The platform handles accounts, synchronized preferences, and hosted services. Local desktop features continue working when the user is signed out, disconnected, or has no active subscription.

| Desktop client | Platform services |
| --- | --- |
| Tray and keyboard screen capture | Account and device management |
| Local OCR | Reading-profile synchronization |
| Customizable reading window | Browser-based profile management and previews |
| Local text-to-speech | Premium cloud voices with defined usage allowances |
| Local settings and offline use | Subscription, billing, and cloud usage dashboard |

## Business model

The business model targets sustainable revenue from hosted services, with pricing informed by customer needs and operating costs.

Start by validating a subscription for premium cloud voices with a monthly allowance. Profile sync can be a free account feature or part of the paid offer; determine that through customer interviews and pricing tests. Institutional plans for schools or teams are a later hypothesis.

Avoid unlimited cloud speech until usage and provider costs are understood. Track paid conversion, recurring revenue, churn, usage per subscriber, and contribution margin:

```text
Contribution per subscriber = net subscription revenue
                            - cloud speech costs
                            - payment processing costs
                            - variable hosting and support costs

Break-even subscribers = fixed monthly operating costs
                       / positive contribution per subscriber
```

Use measured costs and explicitly labeled assumptions in the university business case. Profitability is an objective to validate, not a current claim. Do not fund the service by selling captured text or screenshots.

## Web experience

Build the browser UI with React and JavaScript. The web dashboard should provide:

- Sign-in, account details, connected devices, and session controls.
- Reading-profile editing with an immediate sample-text preview.
- Font, size, spacing, line width, and color controls using the same settings schema as the desktop app.
- Cloud voice selection, explicit sample playback, subscription status, and usage information.
- Billing management and clear behavior when usage limits are reached.
- Feedback submission, support information, and account/data deletion controls.

The preview illustrates preferences; native desktop rendering may differ. Device-specific settings, including the default floating or focus reading mode, remain in the desktop app. All controls need keyboard access, clear labels, visible focus, and understandable loading and error states.

## Architecture

```text
React + JavaScript dashboard         Native C++ desktop app
             |                          platform adapter
             |                               |
             +----------- HTTPS -------------+
                             |
                 Authentication and REST APIs
                             |
               +-------------+--------------+
               |                            |
       Supabase Auth + Postgres       Trusted cloud service
       profiles, devices, settings    entitlements, usage, TTS
                                            |
                                  Speech and billing providers
```

Supabase is the proposed backend platform, with Postgres as the database and Supabase Auth for identity. Its [generated REST API](https://supabase.com/docs/guides/api) can serve authorized preference and device data. Cloud speech, subscription enforcement, and billing webhooks require trusted server-side handlers.

The dashboard uses [React's browser UI APIs](https://react.dev/reference/react-dom). It is deployed separately from the native app; opening a browser is not part of the local capture/OCR path.

Use two repositories: `DyslexiaApp` for the desktop client and `DyslexiaPlatform` for the dashboard and backend. Keep server-side handlers and database migrations alongside the dashboard in this repository; the server implementation language is a separate decision from the JavaScript frontend requirement.

## Accounts, data, and API contracts

Use authenticated HTTPS requests and versioned JSON contracts. Desktop sign-in should open the system browser and return through a validated desktop authentication flow; prototype the callback and token lifecycle before committing to an implementation.

Initial data entities are user profiles, registered devices, reading profiles, subscription entitlements, usage events, and feedback. Captured screenshots and OCR text are not part of settings synchronization.

Enable [row-level security](https://supabase.com/docs/guides/database/postgres/row-level-security) for user-owned data. Clients receive only public project configuration and their own session credentials. Service-role credentials and cloud provider keys stay in trusted server environments.

Users must not be able to grant themselves a paid plan or edit authoritative usage records. Server-side handlers validate entitlement, enforce quotas, and record usage; verified billing webhooks update subscription state. Failed requests and retries need defined metering behavior to avoid duplicate billing.

Document contracts for profile sync, device registration, voice catalogs, speech requests, usage, and errors. Settings revisions support conflict detection. API changes must document compatibility with released desktop versions, and desktop clients must retain local operation when the API is unavailable.

## Proposed nine-person team

These are responsibility assignments to agree as a team, not job titles requiring nine isolated workstreams. Replace member numbers with names. Each area should have a second teammate who can review and maintain it.

| Member | Primary responsibility | Expected ownership |
| --- | --- | --- |
| 1 | Product and client liaison | Requirements, client meetings, priorities, acceptance criteria, business case. |
| 2 | UX and accessibility | User research, reading profiles, interaction designs, accessible workflows, usability sessions. |
| 3 | Desktop shell | C++ tray, shortcuts, native windows, local settings, installation. |
| 4 | Engine and OCR | Engine interfaces, capture/OCR integration, language handling, performance. |
| 5 | Reader and speech | Text rendering, reading customization, local TTS, desktop cloud-provider integration. |
| 6 | Web frontend | React dashboard, interactive settings preview, account and subscription views. |
| 7 | Backend and data | Auth integration, Supabase migrations, access policies, settings sync, API contracts. |
| 8 | Cloud services and billing | Speech provider integration, quotas, billing integration, cost tracking. |
| 9 | Quality and delivery | CI/CD, test coordination, release artifacts, environments, operational checks. |

This table covers the whole project, including work in the desktop repository. Everyone contributes reviews and tests; quality and deployment knowledge must not depend on one teammate.

## Client interaction and project evidence

Maintain an ongoing feedback loop with the project client and representative readers:

1. Agree the problem, target users, initial scope, and measurable acceptance criteria.
2. Demonstrate a working increment each iteration and record feedback, decisions, and owners.
3. Run usability sessions covering capture, floating and focus reading modes, reading customization, speech, settings, and account flows.
4. Turn accepted feedback into tracked issues with clear completion criteria.
5. Obtain client acceptance of milestones and document remaining limitations.

Keep meeting notes, an issue board, decision records, demo evidence, and an acceptance checklist in the project documentation. Obtain permission for recordings and use sample text in demos and reports.

## CI/CD and release ownership

Plan pull-request checks and separate staging and production environments.

| Area | Proposed checks and delivery behavior |
| --- | --- |
| Frontend | Lint, production build, meaningful UI tests, keyboard and accessibility checks. |
| Database | Apply migrations to a disposable database; test user isolation and protected subscription fields. |
| API | Validate requests and contracts; test auth, quota enforcement, retries, and provider failures. |
| Integration | Verify desktop/platform compatibility and continued local use during network failure. |
| Subscription flows | Use provider test modes for subscriptions, webhook verification, cancellations, and limits. |
| Deployment | Deploy reviewed changes to staging, run smoke checks, then promote a known artifact to production. |
| Releases | Publish versioned artifacts, release notes, migration notes, and a recovery procedure. |

The desktop repository owns its own Windows build and packaging pipeline, engine tests, installer artifacts, and signed releases. This repository owns web/API deployments and database migration coordination. Release the two independently while testing their shared contracts.

Keep production secrets out of pull-request jobs, especially external contributions. Prefer compatible migrations so the previously deployed application can keep running during a release. Assign a release owner and reviewer for every production deployment; document how to recover from failed releases and restore data.

## Product and operational roles

Development responsibilities above are separate from application permissions. Begin with ordinary user and restricted administrator roles. Users manage their own preferences and services; administrators handle authorized support and service operations. Limit access to personal data and record privileged changes.

Add organization owner/member roles only if a team or school plan is implemented. Nine developers do not imply nine application permission levels.

## Milestones

1. **Discovery:** client requirements, accessible UI prototype, business assumptions, and API contracts.
2. **Desktop integration baseline:** demonstrate that the desktop works offline and independently.
3. **Account MVP:** authentication, dashboard, profile synchronization, and user-isolation tests.
4. **Cloud services prototype:** one cloud speech provider, subscription test flow, quotas, and a usage/cost dashboard.
5. **University delivery:** client acceptance, CI/CD evidence, usability findings, deployment documentation, and a costed profitability model.

## Decisions still open

- Project branding, source licenses, and named team assignments.
- Initial supported OCR and speech languages, confirmed through the desktop prototype.
- Speech and billing providers, pricing, and monthly allowances.
- Hosting, server-side runtime, and authentication callback design.
- Whether settings sync is free or included in a subscription.

Build and deployment commands will be added once the implementation and tooling are selected.
