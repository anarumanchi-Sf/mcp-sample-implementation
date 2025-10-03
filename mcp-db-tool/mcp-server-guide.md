## Building an MCP Server with Mule 4

This guide explains how to configure a Mule-based MCP server, how to define MCP tools, and how to design robust JSON Schemas for tool parameters. Use this as a template to build your own MCP server and parameter schemas of any complexity.

---

## What You’ll Build

- An MCP server that exposes tools over HTTP
- A tool (`getCustomerDetails`) that fetches data from Oracle DB
- A health check endpoint
- Strongly-typed parameter schemas (JSON Schema Draft-07)
- JSON responses and error handling

---

## Prerequisites

- Java 17
- Maven 3.8+
- Mule Runtime 4.9.x
- An MCP-capable client to invoke tools
- Network access to the target data sources (Oracle in this example)

---

## Project Setup

### Dependencies (POM)
Include the MCP connector and any transport/connectors your tools will use (DB, HTTP, etc.).

```xml
<dependency>
  <groupId>com.mulesoft.connectors</groupId>
  <artifactId>mule-mcp-connector</artifactId>
  <version>1.1.0</version>
  <classifier>mule-plugin</classifier>
</dependency>

<dependency>
  <groupId>org.mule.connectors</groupId>
  <artifactId>mule-db-connector</artifactId>
  <version>1.14.16</version>
  <classifier>mule-plugin</classifier>
</dependency>

<!-- Drivers or other libraries (example: Oracle JDBC) -->
<dependency>
  <groupId>com.oracle.database.jdbc</groupId>
  <artifactId>ojdbc8</artifactId>
  <version>21.9.0.0</version>
</dependency>
```

---

## Configure the MCP Server

1) Create an HTTP listener config
2) Bind the MCP server to the listener
3) Define environment/config properties

```xml
<mule xmlns="http://www.mulesoft.org/schema/mule/core"
      xmlns:http="http://www.mulesoft.org/schema/mule/http"
      xmlns:mcp="http://www.mulesoft.org/schema/mule/mcp"
      xmlns:db="http://www.mulesoft.org/schema/mule/db"
      xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="
        http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd
        http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd
        http://www.mulesoft.org/schema/mule/mcp  http://www.mulesoft.org/schema/mule/mcp/current/mule-mcp.xsd
        http://www.mulesoft.org/schema/mule/db   http://www.mulesoft.org/schema/mule/db/current/mule-db.xsd
        http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd">

  <configuration-properties file="config.properties"/>

  <http:listener-config name="HTTP_Listener">
    <http:listener-connection host="0.0.0.0" port="${http.port}"/>
  </http:listener-config>

  <mcp:server-config name="MCP_Streamable_Config"
                     serverName="${mcp.clientName}"
                     serverVersion="${mcp.clientVersion}">
    <mcp:streamable-http-server-connection listenerConfig="HTTP_Listener"/>
  </mcp:server-config>

  <!-- Example: DB configuration used by a tool -->
  <db:config name="Oracle_DB_Config">
    <db:generic-connection
      url="jdbc:oracle:thin:@//${db.host}:${db.port}/${db.serviceId}?internal_logon=SYSDBA"
      driverClassName="oracle.jdbc.OracleDriver"
      user="${db.user}"
      password="${db.systemPassword}"/>
  </db:config>
</mule>
```

See `src/main/mule/global.xml` in this project for the concrete configuration.

---

## Defining a Tool

A tool is defined with `mcp:tool-listener` inside a Mule flow. It declares:
- Tool metadata (name, description)
- Parameter schema (JSON Schema)
- Response(s) and error responses
- The implementation (Mule processors using the parameters)

Minimal structure:

```xml
<flow name="my-tool-flow">
  <mcp:tool-listener
      doc:name="My Tool"
      config-ref="MCP_Streamable_Config"
      name="myToolName">
    <mcp:description><![CDATA[What this tool does]]></mcp:description>

    <mcp:parameters-schema>
      <![CDATA[
      {
        "$schema": "http://json-schema.org/draft-07/schema#",
        "type": "object",
        "properties": { "example": { "type": "string" } },
        "required": ["example"],
        "additionalProperties": false
      }
      ]]>
    </mcp:parameters-schema>

    <mcp:responses>
      <mcp:text-tool-response-content text="#[write(payload, 'application/json')]"/>
    </mcp:responses>

    <mcp:on-error-responses>
      <mcp:text-tool-response-content text="#[write(payload, 'application/json')]"/>
    </mcp:on-error-responses>
  </mcp:tool-listener>

  <!-- Use parameters via DataWeave expressions or component configs -->
  <!-- e.g., DB/HTTP calls, transformations, aggregations, etc. -->
</flow>
```

In this project, see `src/main/mule/mcp-db-tool.xml` for a complete example.

---

## Parameter Schema Deep Dive

Schemas are standard JSON Schema (Draft-07). They:
- Validate MCP inputs before your flow runs
- Drive documentation and discoverability for MCP clients
- Enable rich UX in schema-aware clients (types, required fields, enums)

### 1) Base pattern

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {},
  "required": [],
  "additionalProperties": false
}
```

- Use "additionalProperties": false to block typos/unknowns.
- Put all fields under "properties", list mandatory ones in "required".

### 2) Strings, numbers, booleans

```json
{
  "type": "object",
  "properties": {
    "customerId": { "type": "string", "minLength": 1 },
    "limit": { "type": "integer", "minimum": 1, "maximum": 100 },
    "activeOnly": { "type": "boolean", "default": true }
  },
  "required": ["customerId"],
  "additionalProperties": false
}
```

- Common string constraints: minLength, maxLength, pattern, format ("email", "date", "date-time", "uuid").
- Numbers: minimum, maximum, exclusiveMinimum, exclusiveMaximum, multipleOf.

### 3) Enums and constants

```json
{
  "type": "object",
  "properties": {
    "status": { "type": "string", "enum": ["NEW", "ACTIVE", "SUSPENDED"] },
    "region": { "const": "US" }
  }
}
```

### 4) Arrays

```json
{
  "type": "object",
  "properties": {
    "tags": {
      "type": "array",
      "items": { "type": "string", "minLength": 1 },
      "minItems": 1,
      "maxItems": 20,
      "uniqueItems": true
    }
  }
}
```

### 5) Nested objects

```json
{
  "type": "object",
  "properties": {
    "address": {
      "type": "object",
      "properties": {
        "street": { "type": "string" },
        "city": { "type": "string" },
        "state": { "type": "string", "minLength": 2, "maxLength": 2 },
        "zip": { "type": "string", "pattern": "^[0-9]{5}$" }
      },
      "required": ["street", "city", "state", "zip"],
      "additionalProperties": false
    }
  }
}
```

### 6) oneOf / anyOf / allOf (unions and composition)

- oneOf: exactly one schema must match (good for XOR shapes)
- anyOf: at least one matches
- allOf: merge/compose multiple

Example: Accept either customerId or (email + dob):

```json
{
  "type": "object",
  "oneOf": [
    { "required": ["customerId"] },
    { "required": ["email", "dateOfBirth"] }
  ],
  "properties": {
    "customerId": { "type": "string" },
    "email": { "type": "string", "format": "email" },
    "dateOfBirth": { "type": "string", "format": "date" }
  },
  "additionalProperties": false
}
```

### 7) Conditional logic (if/then/else)

```json
{
  "type": "object",
  "properties": {
    "includeAddress": { "type": "boolean", "default": false },
    "addressFields": {
      "type": "array",
      "items": { "type": "string", "enum": ["street","city","state","zip","country"] }
    }
  },
  "if": { "properties": { "includeAddress": { "const": true } } },
  "then": { "required": ["addressFields"] },
  "else": { "not": { "required": ["addressFields"] } }
}
```

### 8) Reuse with $defs and $ref

Keep complex shapes DRY and consistent.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$defs": {
    "Pagination": {
      "type": "object",
      "properties": {
        "page": { "type": "integer", "minimum": 1, "default": 1 },
        "pageSize": { "type": "integer", "minimum": 1, "maximum": 100, "default": 25 }
      },
      "additionalProperties": false
    }
  },
  "type": "object",
  "allOf": [
    { "$ref": "#/$defs/Pagination" }
  ],
  "properties": {
    "query": { "type": "string", "minLength": 2 }
  },
  "required": ["query"],
  "additionalProperties": false
}
```

### 9) Patterns for “search” tools

- Optional filters (enums, date ranges)
- Pagination (page, pageSize)
- Sort (sortBy, sortOrder)
- Mutual exclusivity via oneOf or not

```json
{
  "type": "object",
  "properties": {
    "query": { "type": "string", "minLength": 2 },
    "status": { "type": "string", "enum": ["NEW","ACTIVE","SUSPENDED"] },
    "fromDate": { "type": "string", "format": "date" },
    "toDate": { "type": "string", "format": "date" },
    "page": { "type": "integer", "minimum": 1, "default": 1 },
    "pageSize": { "type": "integer", "minimum": 1, "maximum": 100, "default": 25 },
    "sortBy": { "type": "string", "enum": ["createdAt","updatedAt","lastName"] },
    "sortOrder": { "type": "string", "enum": ["asc","desc"], "default": "desc" }
  },
  "allOf": [
    {
      "if": { "required": ["fromDate", "toDate"] },
      "then": {}
    }
  ],
  "additionalProperties": false
}
```

Tips:
- Use "format": "date" or "date-time" for ISO 8601 strings; convert in DataWeave as needed.
- Provide sane defaults (default) to reduce client friction.

---

## Mapping Inputs to Mule Components

- The MCP tool payload is the validated JSON object.
- Access fields using Mule expressions: `payload.customerId`, etc.
- Pass inputs to connectors via input parameters:

```xml
<db:select config-ref="Oracle_DB_Config">
  <db:sql><![CDATA[
    SELECT ... WHERE CustomerID = :customerId
  ]]></db:sql>
  <db:input-parameters><![CDATA[
    #[{ customerId: payload.customerId }]
  ]]></db:input-parameters>
</db:select>
```

See `src/main/mule/mcp-db-tool.xml` for real usage.

---

## Designing Responses

- Use `mcp:text-tool-response-content` to return JSON (or other types).
- Transform to your contract with DataWeave.
- Provide structured error JSON in `mcp:on-error-responses`.

```xml
<mcp:responses>
  <mcp:text-tool-response-content text="#[write(payload, 'application/json')]"/>
</mcp:responses>

<mcp:on-error-responses>
  <mcp:text-tool-response-content text="#[write(payload, 'application/json')]"/>
</mcp:on-error-responses>
```

Error strategy example:
```xml
<error-handler>
  <on-error-propagate logException="true">
    <ee:transform>
      <ee:message>
        <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
  error: error.description default error.message default "Unexpected error"
}]]></ee:set-payload>
      </ee:message>
    </ee:transform>
  </on-error-propagate>
</error-handler>
```

---

## Health Check (Optional but Recommended)

Provide simple HTTP endpoints to validate dependencies:

```xml
<flow name="db-health-check">
  <http:listener config-ref="HTTP_Listener" path="/health/db"/>
  <db:select config-ref="Oracle_DB_Config">
    <db:sql><![CDATA[SELECT 1 AS OK FROM DUAL]]></db:sql>
  </db:select>
  <ee:transform>
    <ee:message>
      <ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{ ok: (payload[0].OK default payload[0]."1") default true }]]></ee:set-payload>
    </ee:message>
  </ee:transform>
</flow>
```

See `src/main/mule/mcp-db-tool.xml` for the full example with error handling.

---

## Security, Config, and Deployment Tips

- Externalize secrets via secure properties (Crypto, Secrets Manager) instead of plain `config.properties`.
- Enable TLS on the HTTP listener for production.
- Restrict network ingress to known MCP clients.
- Apply policies (rate limiting, auth) at your gateway/load balancer.
- For CloudHub 2.0, ensure the inbound ports, workers, and VPC routing allow MCP client access.
- Validate JSON sizes (maxItems, maxLength) to defend against payload abuse.

---

## A Reusable Tool Skeleton

Copy this and replace the internal logic.

```xml
<flow name="tool-<YOUR_NAME>">
  <mcp:tool-listener config-ref="MCP_Streamable_Config" name="<YOUR_NAME>">
    <mcp:description><![CDATA[What the tool does]]></mcp:description>
    <mcp:parameters-schema>
      <![CDATA[
      {
        "$schema": "http://json-schema.org/draft-07/schema#",
        "type": "object",
        "properties": {},
        "required": [],
        "additionalProperties": false
      }
      ]]>
    </mcp:parameters-schema>
    <mcp:responses>
      <mcp:text-tool-response-content text="#[write(payload, 'application/json')]"/>
    </mcp:responses>
    <mcp:on-error-responses>
      <mcp:text-tool-response-content text="#[write(payload, 'application/json')]"/>
    </mcp:on-error-responses>
  </mcp:tool-listener>

  <!-- Implement your logic here using payload.<field> -->
</flow>
```

---

## Local Testing

1) Run the Mule app
2) Ensure the MCP server is listening on `http://localhost:${http.port}`
3) From your MCP client, invoke the tool name (e.g., `getCustomerDetails`) with the schema-compliant JSON parameters
4) Inspect the response payload and logs

---

## Troubleshooting

- Schema validation errors: ensure required fields and formats are correct
- 4xx/5xx from dependencies: log and surface meaningful `error` objects
- Classpath issues (drivers): export needed packages via `mule-artifact.json`
- Connection failures: confirm network, credentials, and firewall rules


