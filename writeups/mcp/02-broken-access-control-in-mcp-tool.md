---
layout: default
title: Broken Access Control in an MCP Server Tool
permalink: /labs/broken-access-control-in-mcp-tool/
---

# Broken Access Control in an MCP Server Tool

## Goal

Use the ordinary `wiener` account to change `carlos`'s email through `update_email`.

## Walkthrough

1. Sign in to the assigned lab as `wiener` and open `/my-account`. Copy the API key shown there; use it only in requests to this lab.
2. In Burp Repeater, initialize the MCP connection with that key as a bearer token. Keep any session header returned by initialization on subsequent requests.
3. Send `tools/list`. Find `update_email` and inspect its input schema. The schema includes both `email` and an optional `username`, which is the important clue: the caller may be able to select the account being changed.

   ```json
   {"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}
   ```

4. Send a `tools/call` request naming `update_email`. Set `username` to `carlos` and supply a valid email address:

   ```json
   {
     "jsonrpc":"2.0",
     "id":3,
     "method":"tools/call",
     "params":{
       "name":"update_email",
       "arguments":{"username":"carlos","email":"attacker@example.com"}
     }
   }
   ```

5. Check the tool result. `Email updated for carlos` means the request was accepted even though the bearer token belongs to `wiener`; the lab should now be solved.

## Why it works

The API key authenticates `wiener`, but the tool trusts a caller-controlled username when selecting the account to modify. The server does not check that the authenticated principal owns or is authorized to update `carlos`.

## Defensive lesson

Derive the target account from the authenticated identity when possible. If a tool accepts an account identifier, authorize that exact object on the server before performing the update.
