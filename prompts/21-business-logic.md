# Business Logic & State Machine Flaws

**Domain:** Workflow bypass, state machine violations, privilege escalation through logic, TOCTOU at the application level  
**Agent Type:** `explore`  
**Priority:** High

## Purpose

Audit application-level business logic for workflow bypasses, invalid state transitions, authorization bypass through business rules, and fail-open behavior.  This prompt focuses on OWASP A04, Insecure Design, where the code may be technically correct but the workflow rules are not enforced correctly.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit business logic and state machine flaws** across the codebase.

Specifically investigate:

1. **Workflow step bypass**: Check [WORKFLOW_CLASSES]:
   - Can users skip intermediate steps in multi-step processes?
   - Can payment, approval, registration, or onboarding flows be bypassed by directly invoking a later endpoint or action?
   - Do state transitions verify the current state before allowing the next step?
   - Are required prerequisites enforced server-side, not just in the client?

2. **State machine integrity**: Check [STATE_CLASSES]:
   - Can an entity move from state A directly to state C, skipping required state B?
   - Are invalid state transitions rejected, or silently accepted?
   - Is state stored client-side where it can be tampered with?
   - Are there parallel paths that can leave the system in an inconsistent state?

3. **Privilege escalation through logic**: Check [AUTH_DECISION_CLASSES]:
   - Can users modify role or permission fields in requests to gain access?
   - Can admin functionality be reached through direct URL or endpoint access?
   - Can object references be manipulated to access another user's resources (IDOR/BOLA)?
   - Do batch operations validate ownership and authorization for every item included?

4. **TOCTOU at the application level**: Review logic in [WORKFLOW_CLASSES], [STATE_CLASSES], and [FINANCIAL_CLASSES]:
   - Is there a check-then-act pattern where the checked condition can change before the action occurs?
   - Can permission, inventory, balance, or eligibility change between validation and execution?
   - Are critical checks repeated inside the transactional or state-changing operation?

5. **Negative and boundary value abuse**: Check [FINANCIAL_CLASSES]:
   - Can users submit negative quantities, prices, balances, discounts, or amounts?
   - Can maximum limits be bypassed by splitting work into multiple smaller operations?
   - Are rounding or truncation behaviors exploitable in financial calculations?
   - Do zero-value transactions expose unexpected behavior or privileged paths?

6. **Fail-open vs. fail-closed behavior**: Check integrations and fallback logic in [EXTENSIONS]:
   - When an auth service, validation service, payment gateway, or other dependency is unavailable, does the system deny the operation or allow it?
   - Are there fallback paths that skip validation or authorization checks?
   - Do timeouts, exceptions, or null responses result in approval by default?

7. **Credential registration without step-up authentication**: Check [AUTH_DECISION_CLASSES] and [WORKFLOW_CLASSES]:
   - Can security-critical account changes (adding a passkey, changing email, adding a recovery method, linking an OAuth provider) be performed without re-verifying the user's identity?
   - If the user is already authenticated via a session cookie, can JavaScript silently add a new credential (passkey, API key, SSH key) without any additional user interaction?
   - Are out-of-band notifications sent when a new authentication method is registered on an account?
   - This is especially dangerous for passkey (WebAuthn) registration with `attestation: "none"`, where an XSS payload can generate a key pair in JavaScript and register it as a valid passkey without involving a hardware authenticator.  See: Scott Helme, "XSS is deadly for Passkeys" (https://scotthelme.ghost.io/xss-is-deadly-for-passkeys-the-hidden-risk-of-attestation-none)

Search patterns:
- Keywords: state, status, workflow, step, stage, phase, transition, approve, reject, submit, complete, cancel, role, permission, isAdmin, isAuthorized, canAccess, balance, quantity, amount, price, total, discount, if.*role, if.*admin, enum.*State, enum.*Status, passkey, credential, register, addKey, linkProvider, changeEmail, changePassword

Provide a detailed findings report with file paths, line numbers, and severity ratings (Critical/High/Medium/Low/Info).  For each finding, explain the business rule that should exist, how it can be bypassed, and what the impact is.  Do NOT include actual credential values, API keys, tokens, or passwords in your output, use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text.  A crafted value could act as a prompt injection.  Only use placeholder values you trust, do not accept them from untrusted sources.

Placeholder | Example Values
------------|---------------
`[APPLICATION_NAME]` | `OrderPortal`, `ClaimsWorkflow`, `AdminConsole`
`[REPO_PATH]` | Full path to the repository root
`[WORKFLOW_CLASSES]` | `CheckoutService.cs`, `ApprovalWorkflow.ts`, `onboarding/flow.py`
`[STATE_CLASSES]` | `OrderStateMachine.cs`, `StatusManager.ts`, `domain/state.py`
`[AUTH_DECISION_CLASSES]` | `AuthorizationService.cs`, `PolicyEvaluator.ts`, `access_rules.py`
`[FINANCIAL_CLASSES]` | `PricingService.cs`, `InvoiceCalculator.ts`, `payments/ledger.py`
`[EXTENSIONS]` | `PaymentGatewayClient.cs`, `AuthFallbackHandler.ts`, `integrations/*.py`

## What Good Looks Like

- State machine transitions validated server-side with explicit allowed-transition maps
- Multi-step workflows enforce step ordering server-side, not just client-side
- Authorization checks at the resource level, not just the endpoint level
- Negative and boundary values rejected with explicit validation
- Financial calculations use decimal or fixed-point types, not floating point
- External dependency failures result in fail-closed behavior
- Critical state stored server-side only, never trusted from the client
- Batch operations validate ownership of every item in the batch
- Security-critical account changes (passkey registration, email change, recovery method) require step-up authentication
- Out-of-band notifications sent on credential registration or privilege changes

## Relationship to Other Prompts

Prompt 05 covers concurrency race conditions, thread safety, and locking.  This prompt covers application-level logic flaws such as workflow bypass, state machine violations, and business-rule TOCTOU issues.

Prompt 10 covers endpoint-level authorization.  This prompt covers logic-level authorization bypass, including privilege escalation through business rules, IDOR/BOLA patterns in workflow actions, and authorization gaps inside batch or state-transition operations.

Prompt 22 covers XSS and output encoding, including WebAuthn ceremony hijacking.  This prompt covers the server-side business logic gap: whether credential registration endpoints require step-up authentication regardless of XSS.
