---
title: "Multi-Tenancy Using Organizations in Auth0"
description: "Learn how to implement multi-tenancy in SaaS applications using Auth0 Organizations. Understand tenant isolation, organization-specific identity providers, authentication flow, and best practices."
pubDate: 2025-06-20
author: "George V. Thomas"
heroImage: '../../assets/blog-auth0-organization.jpg'
tags:
  - Auth0
  - Organizations
  - IAM
  - CIAM
  - Multi-Tenancy
  - OAuth2
  - OpenID Connect
draft: false
---
When building SaaS applications, one of the most important architectural decisions is how to securely isolate customers while providing a seamless authentication experience. Auth0 addresses this requirement through **Organizations**, a feature specifically designed for Business-to-Business (B2B) and multi-tenant applications.

In this article, we'll explore how to implement multi-tenancy using **Organizations in Auth0**, understand the authentication flow, and review some implementation best practices.

---

## Introduction

Imagine a company **Acme** offering a SaaS platform secured using Auth0.

```
https://auth.acme.com
```

As more customers subscribe to the platform, each customer becomes a separate tenant.

Typical requirements include:

- Each customer represents an independent tenant.
- Users must be isolated within their own tenant.
- Each tenant may authenticate using its own Identity Provider (IdP).
- Each tenant may require its own branding and login experience.

Auth0 Organizations provide a structured way to implement this model while using a single Auth0 tenant.

---

## What is an Organization in Auth0?

An **Organization** represents one business customer (tenant) using your SaaS application.

Each organization can have:

- Its own users (members)
- Its own Identity Providers (connections)
- Its own branding (logo, colors, login experience)
- Organization-specific Role-Based Access Control (RBAC)

---

## Auth0 Tenant vs Organization

It's important to distinguish between an **Auth0 Tenant** and an **Organization**.

### Auth0 Tenant

An Auth0 tenant is the top-level container that holds:

- Applications
- APIs
- Identity Providers (Connections)
- Users
- Organizations

### Organization

Organizations exist **inside** an Auth0 tenant and represent your individual business customers.

This enables multiple organizations to share the same Auth0 infrastructure while maintaining logical isolation.

---

## Multi-Tenant Architecture

Consider the following SaaS customers:

- companyA.com
- companyB.com

Each company:

- Has its own identity system
- Wants employees to authenticate using their own corporate credentials

For example, one customer may use:

- Microsoft Entra ID (Azure AD)
- Google Workspace
- SAML Identity Provider
- Another OpenID Connect provider

---

## Architecture

```
                    +----------------------+
                    |    SaaS Application  |
                    +----------+-----------+
                               |
                               |
                    +----------v-----------+
                    |     Auth0 Tenant     |
                    |   auth.acme.com      |
                    +----------+-----------+
                               |
              +----------------+----------------+
              |                                 |
     +--------v--------+               +--------v--------+
     | Organization A  |               | Organization B  |
     |  companyA.com   |               |  companyB.com   |
     +--------+--------+               +--------+--------+
              |                                 |
         Azure AD                         Google Workspace
```

Each organization is configured with its own Identity Providers (Connections).

---

## Authentication Flow

The authentication process typically follows these steps:

1. User accesses the SaaS application.
2. The application redirects the user to Auth0.
3. The authorization request includes an **organization identifier**.
4. Auth0 determines:
   - Which organization the user belongs to.
   - Which Identity Provider should be used.
5. The user is redirected to the appropriate Identity Provider.
6. After successful authentication, Auth0 redirects the user back to the application with OAuth/OIDC tokens.

---

## Passing the Organization Parameter

For multi-tenant applications, the authorization request should include the organization identifier.

```text
organization=<org_id>
```

Example:

```text
https://auth.acme.com/authorize?
client_id=my-app
&response_type=code
&redirect_uri=https://app.acme.com/callback
&organization=org_abc123
```

This tells Auth0 which tenant context should be used during authentication.

---

## Using the Connection Parameter

Organizations can also specify which Identity Provider should be used.

Within the Organization configuration:

**Organizations → Connections**

Enable the required connection and optionally enable **Auto-Membership**, allowing users authenticating through that connection to automatically become members of the organization.

You can then pass:

```text
connection=<connection-name>
```

Example:

```text
https://auth.acme.com/authorize?
client_id=my-app
&response_type=code
&redirect_uri=https://app.acme.com/callback
&organization=org_abc123
&connection=google-oauth2
```

The `connection` parameter allows direct routing to a specific Identity Provider, bypassing the login selection screen.

> If you're familiar with Keycloak, the `connection` parameter is conceptually similar to the `kc_idp_hint` parameter.

---

## Best Practices

For multi-tenant SaaS applications, consider the following recommendations:

- Always pass the `organization` parameter.
- Use the `connection` parameter only when:
  - The Identity Provider is already known.
  - You want to skip the login selection page.
- Configure Identity Providers at the organization level.
- Use organization-specific branding to provide a personalized login experience.
- Enable Auto-Membership where appropriate to simplify user onboarding.

---

## Example Scenario

A user from **companyA.com** accesses the application.

The application redirects the user to:

```text
/authorize?...&organization=org_companyA
```

Auth0 then:

1. Identifies Organization A.
2. Selects the configured Microsoft Entra ID connection.
3. Redirects the user to Microsoft Entra ID.
4. Authenticates the user.
5. Returns the user to the application with OAuth/OIDC tokens.

The resulting ID Token and Access Token include organization-related claims such as:

```json
{
  "org_id": "org_companyA"
}
```

This enables the application to determine which tenant the authenticated user belongs to.

---

## Summary

Auth0 Organizations provide a powerful and scalable approach to implementing multi-tenancy in SaaS applications.

They enable:

- Logical tenant isolation
- Organization-specific Identity Providers
- Tenant-specific branding
- Flexible authentication flows
- Simplified B2B identity management

The combination of:

- **organization** (tenant context)
- **connection** (Identity Provider routing)

provides a clean and flexible architecture for enterprise-grade multi-tenant applications.

---

## References

- Auth0 Organizations Documentation
- OAuth 2.0 Authorization Framework
- OpenID Connect Core Specification