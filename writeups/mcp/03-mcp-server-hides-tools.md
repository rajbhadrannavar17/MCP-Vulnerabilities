---
layout: default
title: MCP Server Hides Tools Without Restricting Them
permalink: /labs/mcp-server-hides-tools/
---

# MCP Server Hides Tools Without Restricting Them

## Goal

Use the lab's staff-only MCP tools to delete the `carlos` account while authenticated as `wiener`.

## Walkthrough

1. Sign in as `wiener`, open `/my-account`, and copy the displayed API key.
2. Initialize `/mcp` with the API key as a bearer token. In Burp Repeater, send `tools/list` and note which tools are advertised.
3. Read the server instructions returned by initialization. They mention `list_users` and `delete_user` as staff tools, even though neither appears in `tools/list`. This is the clue that discovery is hiding the tools rather than the server necessarily enforcing authorization.
4. Try the named `list_users` tool directly using the normal JSON-RPC `tools/call` method:

   ```json
   {
     "jsonrpc":"2.0",
     "id":3,
     "method":"tools/call",
     "params":{"name":"list_users","arguments":{}}
   }
   ```

   A successful result returns the user list. If the server rejects the request, confirm that you reused the initialized session and the `wiener` API key.

5. Call the hidden `delete_user` tool for the lab's target user:

   ```json
   {
     "jsonrpc":"2.0",
     "id":4,
     "method":"tools/call",
     "params":{"name":"delete_user","arguments":{"username":"carlos"}}
   }
   ```

6. The result `Deleted carlos` confirms the operation and completes the lab.

## Why it works

`tools/list` controls what a client discovers, not what the server is willing to execute. The dispatcher accepts a tool name without independently checking the caller's role.

## Defensive lesson

Enforce authorization inside each tool handler or in a shared dispatcher. Never treat omission from discovery as an access-control check.
