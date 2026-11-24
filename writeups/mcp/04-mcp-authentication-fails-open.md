---
layout: default
title: MCP Server Authentication Fails Open
permalink: /labs/mcp-authentication-fails-open/
---

# MCP Server Authentication Fails Open

## Goal

Complete the lab by calling the staff-only delete operation with an invalid bearer token.

## Walkthrough

1. Send an unauthenticated request to the lab's `/mcp` endpoint. The server responds with `401 Unauthorized`; this is the expected baseline.
2. In Burp Repeater, send `tools/list` with a deliberately invalid bearer token:

   ```http
   Authorization: Bearer invalid
   Content-Type: application/json
   ```

   ```json
   {"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}
   ```

3. Compare the response to the unauthenticated baseline. The invalid token is accepted and the server returns a normal MCP response, rather than rejecting it.
4. Keep that same invalid bearer value and call the staff-only tool:

   ```json
   {
     "jsonrpc":"2.0",
     "id":3,
     "method":"tools/call",
     "params":{"name":"delete_user","arguments":{"username":"carlos"}}
   }
   ```

5. The response `Deleted carlos` confirms that the invalid credential passed through to an authorization context with excessive privileges. The lab is solved.

## Why it works

The server treats a missing credential and an unrecognized credential differently. Instead of rejecting an invalid token, it falls back to a context that can still call privileged tools.

## Defensive lesson

Reject missing, malformed, expired, and invalid credentials before dispatching any tool. A failed authentication check must never create a more permissive fallback identity.
