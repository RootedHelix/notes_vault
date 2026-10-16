**IdPs** are **Identity Providers**.

An **Identity Provider (IdP)** is a service whose **only job** is to **prove who a user or client is** and **issue trusted identity assertions (tokens)** that other systems rely on.

In OAuth/OIDC terms, an IdP is usually the **Authorization Server** (or the core of it).

---

## Plain-English definition

> **An Identity Provider is the authority that authenticates users and issues tokens saying “this identity is verified.”**

Everything else (APIs, gateways, services) **trusts the IdP** instead of handling identity itself.

---

## What an IdP actually does

An IdP handles **identity-heavy responsibilities** that most systems should _not_ re-implement:

### Authentication

- Username/password
    
- MFA (TOTP, SMS, hardware keys)
    
- Passkeys / WebAuthn
    
- SSO / federation (Google, GitHub, enterprise AD)
    

### Authorization inputs

- User roles
    
- Groups
    
- Claims
    
- Scopes
    

### Token issuance

- OAuth 2.0 access tokens
    
- OpenID Connect ID tokens
    
- Refresh tokens
    
- Signed JWTs
    
- Key rotation (JWKS)
    

### User-facing UI

- Login pages
    
- Consent screens
    
- Account management
    
- Password reset flows
    

---

## What an IdP does **not** do

An IdP does **not**:

- Serve your application data
    
- Run business logic
    
- Handle inference
    
- Proxy requests
    
- Enforce rate limits on APIs
    

Those belong to **Resource Servers / gateways** (like your C server).

---

## How IdPs fit into the system (lock this in)

```
User / Client
     ↓
Identity Provider (IdP)
  (login, MFA, consent)
     ↓
Signed Token (JWT)
     ↓
Resource Server / Gateway
 (verify token, enforce policy)
     ↓
Protected resource
```

The IdP is the **source of truth for identity**.

---

## Common real-world IdPs (examples)

These are all **IdPs / Authorization Servers**:

- **Auth0** – SaaS IdP
    
- **Okta** – enterprise IdP
    
- **Keycloak** – self-hosted IdP
    
- **Azure Active Directory** – Microsoft IdP
    
- **Google Identity** – federated IdP
    

They all:

- authenticate users
    
- issue tokens
    
- publish public keys
    
- follow OAuth/OIDC standards
    

---

## Why IdPs exist (this is the key insight)

Identity is:

- extremely security-sensitive
    
- full of edge cases
    
- constantly evolving
    
- legally and operationally risky
    

So the industry learned:

> **Centralize identity. Don’t reinvent it per service.**

IdPs let:

- security teams harden one place
    
- application teams stay productive
    
- infrastructure stay fast and simple
    

---

## How this maps to _your_ architecture

In your world:

- **IdP** (Go / Java / SaaS)
    
    - authenticates users
        
    - issues JWTs
        
- **Your C server**
    
    - validates JWTs
        
    - enforces scopes / quotas
        
    - streams responses
        
    - protects expensive resources (LLMs, hardware)
        

This is exactly how modern systems are built.

---

## One-sentence takeaway

> **An IdP is a dedicated identity authority that authenticates users and issues trusted tokens so the rest of your system doesn’t have to deal with identity at all.**

If you want next, I can:

- show how a C resource server trusts an IdP step by step
    
- explain IdP vs OAuth vs OIDC without jargon
    
- help you choose between hosted vs self-hosted IdPs
    
- map IdPs cleanly onto embedded or air-gapped systems