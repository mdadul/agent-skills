# Proxy Examples

## 1) Cached YouTube Service

### Context
Third-party client repeatedly requests same video metadata and lists.

### Design
- Service interface: `ThirdPartyYouTubeLib`.
- Real service: `ThirdPartyYouTubeClass`.
- Proxy: `CachedYouTubeClass` adds caching and reset lifecycle.

### Outcome
Client behavior stays unchanged while bandwidth and latency drop on repeated calls.

## 2) Virtual Proxy for Heavy Report Engine

### Context
Report engine startup is expensive but needed only for report endpoints.

### Pattern Use
- Proxy holds nullable engine reference.
- Initializes engine on first call and reuses instance thereafter.
- Optionally disposes on inactivity timeout.

### Outcome
Application startup remains fast without changing report client code.

## 3) Protection Proxy for Admin Operations

### Context
Service exposes sensitive operations and must enforce role-based policy.

### Pattern Use
- Proxy checks credentials/claims before delegation.
- Denied access returns stable authorization errors.
- Audit logs emitted for sensitive method calls.

### Outcome
Authorization is centralized at service boundary, not duplicated across callers.

## 4) Quick Evaluation Prompts
- "Evaluate whether this boundary needs Proxy, Decorator, or Adapter."
- "Refactor this service integration to a lazy-loading virtual proxy."
- "Design a caching proxy with invalidation rules for this API."
- "Create a protection proxy plan that preserves service interface compatibility."
