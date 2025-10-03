# MCP DB Tool (Mule 4)

A Mule 4 application that exposes a Model Context Protocol (MCP) server tool to retrieve customer details from an Oracle database, plus a lightweight HTTP health check for database connectivity.

## Overview

- **Runtime**: Mule 4.9.x (Java 17)
- **Artifact**: `mcp-db-tool` (packaged as a Mule application)
- **Key Connectors**:
  - MCP Connector `mule-mcp-connector:1.1.0`
  - Database Connector `mule-db-connector:1.14.16`
  - Oracle JDBC Driver `ojdbc8:21.9.0.0`
- **Main flow**: `mcp-get-customer-details-flow`
- **Health check**: `GET /health/db`

## Architecture

- `src/main/mule/global.xml`
  - Loads `config.properties`
  - `HTTP_Listener` on `0.0.0.0:${http.port}`
  - `MCP_Streamable_Config` using the HTTP listener
  - `Oracle_DB_Config` with a generic JDBC connection to Oracle

- `src/main/mule/mcp-db-tool.xml`
  - `mcp:tool-listener` named `getCustomerDetails` accepting a `customerId`
  - Performs a `db:select` against `MYDB.CUSTOMER`
  - Transforms the first row into a normalized JSON response
  - Error handler returns JSON error payloads
  - `db-health-check` flow provides an HTTP JSON ping to validate DB connectivity

- `src/main/resources/config.properties`
  - Oracle connection properties and MCP/HTTP configuration

- `mule-artifact.json`
  - Exports Oracle JDBC packages for runtime classloading (min Mule 4.9.9)

## MCP Tool Contract

- **Tool name**: `getCustomerDetails`
- **Parameters**:
  - `customerId` (string, required)
- **Success response**: JSON object including `customerId`, `firstName`, `lastName`, `email`, `phoneNumber`, `dateOfBirth`, `address{...}`, `createdAt`, `updatedAt`
- **Not found**: `{ "error": "Customer not found", "customerId": "<input>" }`
- **Error**: `{ "error": "Unable to retrieve customer details" }`

## HTTP Health Check

- `GET /health/db` responds with:
  - Success: `{ "ok": true }` (based on `SELECT 1 FROM DUAL`)
  - Failure: `{ "error": "<details>" }`

## Prerequisites

- Java 17
- Maven 3.8+
- Mule 4.9.x runtime
- Network access to the target Oracle DB

## Configuration

Edit `src/main/resources/config.properties`:

- `db.host`, `db.port`, `db.user`, `db.systemPassword`, `db.serviceId`
- `mcp.clientName`, `mcp.clientVersion`
- `http.port`

Note: Credentials in `config.properties` are for development; use secure properties or externalized secrets for non-dev.

## Build

```bash
mvn -q -f mcp-db-tool/pom.xml clean package
```

The packaged application will be created under `mcp-db-tool/target/`.

## Run Locally

- Start Mule runtime with this application or use Studio.
- Ensure Oracle DB is reachable from your environment.
- Verify:
  - MCP listener started (tool `getCustomerDetails` available)
  - Health: `curl http://localhost:${http.port}/health/db`

## Using the MCP Tool

From an MCP-capable client, invoke tool `getCustomerDetails` with parameters:

```json
{ "customerId": "12345" }
```

The response will be a JSON object with customer details or an error.

## Logging

- Configured via `src/main/resources/log4j2.xml`
- Default root level: `INFO`
- App logger: `org.mule.mcp-db-tool` at `DEBUG`

## Notes

- The DB query targets schema/table `MYDB.CUSTOMER` and expects columns:
  `CustomerID, FirstName, LastName, Email, PhoneNumber, DateOfBirth, StreetAddress, City, State, ZipCode, Country, CreatedAt, UpdatedAt`.
- `mule-artifact.json` exports Oracle packages needed by the driver in CloudHub/Runtime Fabric.


