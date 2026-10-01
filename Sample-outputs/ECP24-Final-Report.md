I'll analyze ticket ECP-24 and prepare the full engineering report.

# Engineering Defect Resolution Report — ECP-24

**Report Generated:** 30 September 2026
**Status of Findings:** Root cause unconfirmed (confidence ~0.88); all fixes **Proposed**

---

## 1. Executive Summary

ECP-24 ("JWT token validation failing") reports that authenticated API requests are rejected with unauthorized errors. It is a **Highest** priority ticket under Epic **ECP-19 User Authentication & Security**, classified as an **Authentication Defect** of **Critical** severity affecting the **Authentication** module.

Business impact is total: logged-in users are blocked from account, checkout and order operations, producing complete loss of authenticated session usability, session drop-offs, abandoned transactions and revenue loss.

Historical analysis of 10 resolved defects returns an aggregate relevance of **HIGH (0.92)**, dominated by **ECP-20** (relevance 0.95), whose root cause was a **JWT secret key configuration mismatch between authentication service instances** (generation and validation using different secrets), resolved by unifying JWT configuration across all deployment environments.

The identified **primary root cause** for ECP-24 is a **JWT secret key / signing configuration mismatch across authentication service instances and environments** — tokens issued under one signing secret/algorithm cannot be verified by the validating instance, so signature verification fails and every authenticated request is rejected. Confidence is **High (~0.88)**; it is not higher because expiry-related causes (ECP-22, 0.82) cannot be excluded without validation-layer error detail (signature-invalid vs token-expired) and instance-level configuration comparison.

Six fixes are proposed: **FIX-01** (unify JWT signing secret and algorithm — Primary, Configuration, Medium risk), **FIX-02** (fail-fast startup self-check and deployment configuration-fingerprint gate — Primary, Deployment, Low risk), plus supporting **FIX-03** (failure-mode differentiation), **FIX-04** (observability), **FIX-05** (header/claim validation and UTC clock-skew handling) and optional hardening **FIX-06** (kid-based verification with bounded key allow-list, Medium risk).

Source-code investigation is tightly scoped by a strong repository signal: **only 3 files import `jsonwebtoken`** and **`JWT_SECRET` appears in exactly 3 locations**. A 17-step investigation order begins at `ConfigManager` in `packages/core/framework/src/config/config.ts`.

---

## 2. Ticket Details

| Attribute | Value |
|---|---|
| Ticket ID | ECP-24 |
| Title | JWT token validation failing |
| Description | Authenticated API requests are rejected with unauthorized errors. |
| Priority | Highest |
| Epic | ECP-19 — User Authentication & Security |

**Data completeness note:** the source record contains **only these four attributes**. Reporter, assignee, status, dates, environment, components, steps to reproduce, logs, comments and attachments are **not present**. The ticket is **not marked DONE** and contains **no Root Cause or Resolution section**. Consequently, failure-onset timing and error-type discrimination are unavailable from the ticket itself.

---

## 3. Defect Classification

| Field | Value |
|---|---|
| Category | Authentication Defect |
| Severity | Critical |
| Affected Module | Authentication |

**Business Impact:** Authenticated API requests are rejected with unauthorized errors, blocking logged-in users from account, checkout and order operations. This constitutes complete loss of authenticated session usability, leading to session drop-offs, abandoned transactions and revenue loss.

**Keywords:** jwt, token, validation, unauthorized, authentication, session, api, security.

---

## 4. Historical Analysis

**Source:** Historical_Resolved_Bugs.docx — 10 resolved defects reviewed: ECP-2, ECP-3, ECP-8, ECP-9, ECP-14, ECP-20, ECP-22, ECP-26, ECP-30, ECP-31.
**Aggregate relevance: HIGH (0.92).** No other authentication-module tickets exist in the archive beyond those listed below.

| Ticket | Title | Priority | Module | Relevance | Root Cause | Resolution |
|---|---|---|---|---|---|---|
| ECP-20 | User login failing with valid credentials | Highest | Authentication | **0.95** | JWT secret key configuration mismatch between authentication service instances (generation and validation used different secrets) | Unified JWT configuration across all deployment environments; updated auth service config; verified login |
| ECP-22 | User session expires unexpectedly | High | Authentication / Session | **0.82** | Session timeout incorrectly configured to 5 minutes via environment configuration override | Set timeout to 30 minutes, removed conflicting env override, verified across browsers |
| ECP-9 | Coupon code not applied | High | Promotions validation service | **0.55** | Timezone conversion issue in expiration date comparison marking active items expired | Normalized timezone handling, corrected expiration comparison, verified across regions |
| ECP-2 | Credit card payment fails during checkout | Highest | Payment | **0.45** | Token became null when VISA validation service timed out; workflow processed without validating token, returning 500 | Null validation before processing, retry logic, improved timeout handling |

**Interpretation:** ECP-20 is a near-exact precedent for ECP-24 and directly informs the primary root cause. ECP-22 supplies the leading alternative (lifetime misconfiguration via env override), ECP-9 the timezone/clock-skew alternative, and ECP-2 the null-token alternative.

---

## 5. Repository Analysis

**Source:** Medusa_Auth_Module.docx

- **Module:** Authentication — folder **`Packages/modules/auth`**
- **Responsibilities:** User Authentication, Login, Session Management, **JWT Validation**, Authorization
- **Related components:** JWT token validation/verification logic; login handler; session management (lifecycle, timeout); authorization layer; auth configuration (**JWT secret**); account registration and password reset flows
- **Common failure areas:** token expiration, **JWT secret mismatch**, session timeout, authentication validation errors
- **Business responsibility:** users cannot access the platform when authentication fails
- **Module's related Jira bugs:** JWT token validation failing (**ECP-24**), user login failing with valid credentials, password reset email not received, user session expires unexpectedly, account registration throws validation error

The module's documented common-failure profile lists JWT secret mismatch explicitly, aligning the repository evidence with the ECP-20 historical precedent.

---

## 6. Source Code Investigation Targets

**Source:** Medusa_Source_Code_Context_Document.docx

### 6.1 Confirmed Repository Evidence

- Only **3 files import `jsonwebtoken`**:
  - `packages/modules/auth/src/providers/medusa-cloud-auth.ts`
  - `packages/modules/providers/auth-google/src/services/google.ts`
  - `packages/modules/user/src/services/user-module.ts`
- **`JWT_SECRET` appears in exactly 3 locations**: `framework/src/config/config.ts` (×1) and `utils/src/common/define-config.ts` (×2).
- `packages/modules/auth` contains **0 repository files** (7 models, 8 migrations) — **no repository-layer investigation applies**.

### 6.2 Primary Module

`Packages/modules/auth` — **30 source files**: migrations 8, models 7, utils 6, services 5, providers 4, loaders 2, root 2, types 1.

### 6.3 Repository Paths

| Path | Detail |
|---|---|
| `packages/modules/auth` | 30 source files (breakdown above) |
| `packages/core/framework/src/config/config.ts` | ConfigManager, 212 lines, `JWT_SECRET` / `JWT_PUBLIC_KEY` / `COOKIE_SECRET` |
| `packages/core/utils/src/common/define-config.ts` | 2 of the 3 `JWT_SECRET` occurrences |
| `packages/core/framework/src/config/loader.ts` | configLoader, 54 lines |
| `packages/core/framework/src/http/express-loader.ts` | session cookie security, SameSite/Secure, Redis sessions |
| `packages/core/framework/src/http/utils/define-middlewares.ts` | 57 lines |
| `packages/core/framework/src/http/utils/policies/rbac-field-filter.ts` | RBACFieldFilter, 487 lines |
| `packages/medusa/src/api/auth/[actor_type]/providers/route.ts` | 36 lines |
| `packages/medusa/src/api/admin/users/[id]/auth-providers/route.ts` | 38 lines |
| `packages/core/core-flows/src/auth/workflows/index.ts` | auth workflows barrel |
| `packages/core/types/src/auth/` | 16 files |
| `packages/core/utils/src/auth/` | 4 files |
| `packages/admin/admin-bundler/src/utils/config.ts` | `ADMIN_JWT_TOKEN_STORAGE_KEY`, `ADMIN_AUTH_TYPE` |

### 6.4 Candidate Services

| Service | Path | Lines |
|---|---|---|
| AuthModuleService | `packages/modules/auth/src/services/auth-module.ts` | 1261 |
| AuthProviderService | `.../services/auth-provider.ts` | 170 |
| AuthVerificationProviderService | `.../services/verification-provider.ts` | 74 |
| AuthMfaProviderService | `.../services/mfa-provider.ts` | 131 |
| UserModuleService | `packages/modules/user/src/services/user-module.ts` (imports `jsonwebtoken`) | 392 |
| ConfigManager | `packages/core/framework/src/config/config.ts` | 212 |
| services barrel | `services/index.ts` | 5 |

### 6.5 Candidate Classes

AuthModuleService; AuthProviderService; AuthVerificationProviderService; AuthMfaProviderService; **MedusaCloudAuthService** (`providers/medusa-cloud-auth.ts`, 278 lines, imports `jsonwebtoken` + `jwks-rsa`); TotpMfaProvider (`providers/mfa/totp.ts`, 254 lines); ConfigManager; UserModuleService; RBACFieldFilter; GoogleAuthService; OidcAuthService; GithubAuthService; EmailPassAuthService; Auth (`packages/core/js-sdk/src/auth/index.ts`).

### 6.6 Candidate Models

`auth-identity.ts`, `provider-identity.ts`, `auth-verification.ts`, `auth-password-reset-token.ts`, `auth-mfa-factor.ts`, `auth-mfa-recovery-code.ts`, `models/index.ts`, `user.ts`, `api-key.ts` (TokenDTO).

### 6.7 Candidate Providers

| Provider | Priority / Note |
|---|---|
| MedusaCloudAuthService | **HIGHEST** — only `jsonwebtoken` + `jwks-rsa` consumer in the auth module |
| GoogleAuthService | `@medusajs/auth-google` |
| OidcAuthService | `@medusajs/auth-oidc` — 7 files, `engine/`, claim mappings |
| EmailPassAuthService | — |
| GithubAuthService | — |
| TotpMfaProvider | Best place to confirm or eliminate clock skew |
| Provider loaders | `packages/modules/auth/src/loaders/providers` (2 loader files) |

### 6.8 Recommended Investigation Order (17 steps)

1. `config.ts` — ConfigManager `JWT_SECRET` / `JWT_PUBLIC_KEY` resolution
2. `define-config.ts` — defaults and overrides
3. `medusa-cloud-auth.ts` — `jwt.verify` options (algorithm, kid, issuer, audience, clockTolerance)
4. `auth-module.ts` — `authenticate` → `validateCallback` → token issuance
5. `auth-provider.ts` — provider resolution returns populated token
6. `verification-provider.ts` — timeout / null-token path
7. `utils/verification-token.ts` + utils (6 files) — TTL arithmetic, s vs ms, UTC vs local
8. `totp.ts` + `utils/totp`, `utils/mfa` — clock skew
9. `user-module.ts` — secret / `expiresIn` comparison
10. `express-loader.ts` — session / cookie
11. `define-middlewares.ts` — middleware order
12. Reproduce 401 on concrete routes; capture Authorization header and decoded claims
13. `core/types/src/auth/providers/*.ts` — claim mappings
14. `auth-google` + `auth-oidc` engine — issuer / JWKS cross-check
15. `rbac-field-filter.ts` — if failure is 403-flavoured
16. Migrations (8) and models (7) — schema drift
17. `admin-bundler` `config.ts` — stale or malformed header

> **Distinction:** Section 6.1 items are **Confirmed Repository Evidence** (measured counts and import locations). Sections 6.2–6.8 are **Investigation Targets** — candidate surfaces to examine, not confirmed defect sites.

---

## 7. Root Cause Analysis

### 7.1 Primary Root Cause — Confidence High (~0.88)

**JWT secret key / signing configuration mismatch across authentication service instances and environments.** Tokens issued with one signing secret/algorithm cannot be verified by the validating instance, so signature verification fails and every authenticated request is rejected as unauthorized.

**Confidence drivers:** ECP-20 precedent (relevance 0.95); the Authentication module's documented failure profile (JWT secret mismatch); and the blanket, all-authenticated-requests nature of the symptom.
**Confidence ceiling:** not higher than ~0.88 because expiry-related causes (ECP-22, 0.82) cannot be excluded without validation-layer error detail (signature-invalid vs token-expired) and instance-level configuration comparison.

**Reasoning:** Uniform rejection immediately after successful authentication points to the verification step (signature/secret) rather than credential handling. This is likely a recurrence or incomplete rollout of the ECP-20 unification — for example, an instance or environment still holding the old secret, or a rotation applied only to the issuer and not the validator.

### 7.2 Alternative Root Causes

| # | Alternative | Probability | Precedent | Distinguishing Signal |
|---|---|---|---|---|
| 2 | Token expiration / session lifetime misconfiguration via env override | Medium | ECP-22 (0.82) | Failures begin only after a short interval per session |
| 3 | Timezone / clock-skew defect in `exp`/`nbf`/`iat` comparison | Low-Medium | ECP-9 (0.55) | Failures correlate with a fixed offset or specific regions/instances |
| 4 | Null/absent token at validation from upstream validation-service timeout with no null guard | Low | ECP-2 (0.45) | Intermittent, load-correlated, timeout/null-reference errors |
| 5 | Token propagation/format regression — malformed Authorization header, missing `Bearer` prefix, truncated token, issuer/audience mismatch | Low | — | Malformed-token / claim-mismatch errors scoped to specific clients or routes |

---

## 8. Recommended Fixes

**Status of all fixes: Proposed.**

| ID | Type | Area | Risk | Summary |
|---|---|---|---|---|
| FIX-01 | Primary — Configuration | `Packages/modules/auth` JWT signing/verification config resolution | **Medium** | Unify the JWT signing secret and algorithm across every auth service instance and environment; collapse signing and verification into one resolution path; remove per-instance/per-environment overrides and silent default/fallback secrets; roll out to all instances simultaneously. Supported by historical precedent (ECP-20). |
| FIX-02 | Primary — Deployment | Service startup / config bootstrap | Low | Startup self-check that fails fast when the JWT secret is absent, empty, below required strength, or when signing and verification algorithms differ; deployment gate comparing a one-way configuration fingerprint (`keyId\|algorithm\|hash(secret)`) across all instances on an internal-only diagnostic surface before traffic admission. |
| FIX-03 | Supporting — Error Handling | Token validation path | Low | Differentiate internal failure modes — signature-invalid, expired, not-yet-valid, malformed/unparseable, missing token, issuer/audience mismatch — with a null/empty guard before parsing; keep the external response generic. |
| FIX-04 | Supporting — Observability | Validation telemetry | Low | Counter of validation failures dimensioned by reason code, key id, algorithm, instance, environment; alert on sustained signature-invalid rate and on sharp drop in authenticated-request success rate; never log token, signature, secret or full claims; propagate correlation id. |
| FIX-05 | Supporting — Validation | Header parsing and claim validation | Low | Require a well-formed Authorization header (case-insensitive `Bearer` scheme, trimmed, single token segment); reject structurally malformed tokens before verification; explicitly validate issuer and audience; explicit configured clock-skew tolerance with all `exp`/`nbf`/`iat` comparisons in UTC; TTL sourced from the unified config. |
| FIX-06 | Optional Hardening — Security | Key management | **Medium** (requires source verification) | Key-identifier-based verification with a bounded allow-list of current + immediately-previous keys for zero-downtime rotation; strict algorithm allow-list rejecting unexpected or `none` algorithms. |

**Implementation guidance** consists of two pseudocode blocks: (a) `resolveSigningConfig` / `configFingerprint` with fail-fast startup; and (b) a staged `validateRequestToken` — null guard → structural checks → algorithm allow-list → key lookup by `kid` → signature check → UTC temporal checks → issuer/audience checks, each emitting a distinct reason code.

### 8.1 Configuration Checks (6)

1. Resolved JWT signing secret is byte-for-byte identical across all auth service instances; compare the config fingerprint between issuing and validating instances.
2. Signing algorithm at issuance matches the algorithm expected at verification; no instance falls back to a default/placeholder secret.
3. No environment-specific or instance-local override silently replaces the unified JWT config, including deploy-time-only overrides.
4. Runtime token lifetime / session TTL matches the intended value and is not shortened by an env override (precedent ECP-22: **5 minutes instead of 30**).
5. Clock-skew tolerance is explicitly configured, all instances are time-synced to a common source, and `exp`/`nbf`/`iat` are compared in UTC (precedent ECP-9).
6. Expected issuer and audience at the verifier match the values stamped by the issuer.

### 8.2 Security Considerations

- Never log or emit the JWT secret, raw token or signature; telemetry limited to key id, algorithm name and reason code.
- Keep external authentication-failure responses generic.
- Enforce a strict algorithm allow-list; reject `none` and unexpected algorithms **before** verification (algorithm-substitution defence).
- The secret must meet the algorithm's strength requirement and live in the managed secret store — not in source control or plain config.
- Treat this as a potential secret-exposure trigger: if the mismatch arose from an unplanned or partial rotation, plan a controlled rotation rather than permanently reverting all instances to the old secret.
- Ensure the config-fingerprint diagnostic surface is unreachable from untrusted networks.

---

## 9. Deployment Considerations

**Deployment**

- Roll FIX-01 out to **all instances simultaneously**; a partial rollout reproduces the mismatch.
- Any secret change **invalidates tokens issued under the previous secret** — plan for forced re-authentication, or stage the change behind **FIX-06** bounded multi-key verification to avoid mass session invalidation.
- **Sequence:** verification must accept both the previous and the new key **before** issuance switches to the new key.
- Gate traffic admission on the FIX-02 configuration-fingerprint comparison across all instances (internal-only diagnostic surface).
- **Capture the pre-change configuration fingerprint of every instance before deployment.**

**Rollback**

- Roll back unified JWT configuration **only as a coordinated all-instance operation** — partial rollback recreates the mismatch.
- On rollback, **reverse the key sequence** used during rollout (retire issuance of the new key before narrowing verification).
- Keep the fail-fast startup check **separable** so it can be disabled independently in an emergency — noting that doing so removes the safety gate.
- **FIX-03, FIX-04 and FIX-05 are behaviour-preserving for valid tokens and independently reversible.**

---

## 10. Risks and Assumptions

### 10.1 Risk Assessment

| Risk area | Level | Basis |
|---|---|---|
| Current production impact | **Critical** | Complete loss of authenticated session usability; account, checkout and order operations blocked; revenue loss |
| FIX-01 change risk | **Medium** | Touches shared JWT signing/verification configuration; must be applied to all instances at once |
| FIX-06 change risk | **Medium** | Key-management change; requires source verification and compatible token format |
| FIX-02 / FIX-03 / FIX-04 / FIX-05 change risk | **Low** | Behaviour-preserving for valid tokens; independently reversible |
| Mass session invalidation | **High if unmitigated** | Any secret change invalidates previously issued tokens unless staged behind FIX-06 |
| Diagnosis certainty | **Moderate** | Primary root cause at ~0.88; alternatives 2–5 remain open |
| Evidence gap | **High** | Ticket lacks environment, logs, reproduction steps and timestamps |
| Secret-exposure exposure | **Conditional** | If mismatch stems from an unplanned/partial rotation, a controlled rotation is required |

### 10.2 Assumptions to Validate

1. That issuance and verification configuration paths in `Packages/modules/auth` **can currently diverge** — needs source verification.
2. That a **single authoritative config source** for JWT signing material is available to all instances in this deployment model.
3. That the **ECP-20 resolution applies to the same configuration surface** as ECP-24 — supporting precedent only, **not proof**.
4. That the validation path does **not already** implement null guards, issuer/audience checks or explicit clock-skew tolerance — confirm before adding duplicate logic.
5. That **key-identifier-based verification is compatible** with the currently issued token format — tokens lacking a `kid` need an issuance change first.
6. That **alternative root causes remain open** — the primary cause is unconfirmed at ~0.88, and no proposed change should be treated as the confirmed fix.
7. That the **ticket record's missing data** (environment, logs, reproduction steps, timestamps) leaves failure-onset timing and error-type discrimination unavailable until gathered.

---

## 11. Next Actions

1. **Complete the ticket record** — obtain environment, logs, reproduction steps and timestamps for ECP-24; specifically capture whether the validation layer reports *signature-invalid* versus *token-expired*, as this single signal separates the primary root cause from Alternative 2.
2. **Execute the 17-step investigation order**, beginning with Step 1 (`config.ts` ConfigManager `JWT_SECRET` / `JWT_PUBLIC_KEY` resolution) and Step 2 (`define-config.ts` defaults/overrides) — the 3 confirmed `JWT_SECRET` locations make this the shortest path to confirmation.
3. **Inspect the HIGHEST-priority provider** `packages/modules/auth/src/providers/medusa-cloud-auth.ts` (278 lines, sole `jsonwebtoken` + `jwks-rsa` consumer) for `jwt.verify` options: algorithm, kid, issuer, audience, clockTolerance (Step 3).
4. **Run the six Configuration Checks** (Section 8.1), starting with a fingerprint comparison between issuing and validating instances.
5. **Confirm or eliminate alternatives** using their distinguishing signals — per-session short-interval onset (Alt 2), fixed offset/region correlation (Alt 3), load-correlated intermittency (Alt 4), client/route-scoped malformed-token errors (Alt 5).
6. **Validate the seven assumptions** in Section 10.2 against source before implementing FIX-01, FIX-05 or FIX-06.
7. **Sequence the remediation:** land FIX-02, FIX-03, FIX-04 and FIX-05 (Low risk, reversible) to establish diagnostic signal, then execute FIX-01 as a coordinated all-instance rollout, staged behind FIX-06 if mass session invalidation must be avoided.
8. **Capture pre-change configuration fingerprints** of every instance and confirm the coordinated rollback plan before any deployment window opens.
9. **Assess secret-exposure posture** — if the mismatch traces to an unplanned or partial rotation, schedule a controlled rotation instead of reverting all instances to the old secret.
10. **Re-verify and close** — confirm authenticated API requests succeed across all instances and environments, then record Root Cause and Resolution on ECP-24 (currently absent) before marking DONE.