---
layout: default
title: MCP Session ID Used for Authentication
permalink: /labs/mcp-session-id-used-for-authentication/
---

# MCP Session ID Used for Authentication

## Goal

Use the lab's predictable MCP session identifiers to find an administrator session and delete `carlos`.

## Walkthrough

1. Sign in as `wiener`, open `/my-account`, and copy the API key.
2. Initialize `/mcp` with the API key. Record the `Mcp-Session-Id` returned by the lab; the example response uses a short numeric value. Reuse this session for normal requests so you can see the expected header and response format.
3. Send `tools/list` and confirm the server is responding to MCP requests. The lab's flaw is that the session identifier itself is short and predictable, and the server uses it to select a privileged session.
4. In Burp Repeater, call `list_users` with no bearer token and replace `Mcp-Session-Id` with candidate values from the lab's small session range. Test candidates manually within this lab; do not send these requests to any other service.

   ```http
   Content-Type: application/json
   Mcp-Session-Id: <candidate-session-id>
   ```

   ```json
   {
     "jsonrpc":"2.0",
     "id":10,
     "method":"tools/call",
     "params":{"name":"list_users","arguments":{}}
   }
   ```

5. Compare the tool results. The administrator's session can list `administrator`, `wiener`, and `carlos`; ordinary or invalid session IDs do not return that privileged result. Record the candidate that produces the administrator result.
6. Replace `list_users` with `delete_user`, keep the discovered session ID, and target `carlos`:

   ```json
   {
     "jsonrpc":"2.0",
     "id":11,
     "method":"tools/call",
     "params":{"name":"delete_user","arguments":{"username":"carlos"}}
   }
   ```

7. `Deleted carlos` confirms the lab is solved.

## Why it works

The server treats possession of a session ID as proof of identity, but the ID is predictable. A session identifier should only reference server-side state; it must not replace authentication or per-request authorization.
