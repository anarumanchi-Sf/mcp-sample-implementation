## MCP DB Tool - AI Assistant Prompts

Use these prompts step-by-step to generate the same Mule application, configure it, build, and deploy to CloudHub.

### Prompt A (Project + POM)
Create a Mule 4 (4.9.9, Java 17, mule-maven-plugin 4.3.0) project named mcp-db-tool. Add dependencies: mule-mcp-connector 1.1.0 (mule-plugin), mule-db-connector 1.14.16 (mule-plugin), ojdbc8 21.9.0. Create mule-artifact.json with minMuleVersion 4.9.9 and Java 17.

### Prompt B (Global config)
In src/main/mule/global.xml, add configuration-properties for config.properties. Add MCP client-config using streamable-http-client-connection with placeholders for mcp.clientName, mcp.clientVersion, mcp.serverUrl. Add DB config with oracle-connection using db.host, db.port, db.user, db.systemPassword, db.serviceId.

### Prompt C (Flow)
In src/main/mule/mcp-db-tool.xml, create flow mcp-get-customer-details-flow with mcp:tool-listener name=getCustomerDetails, parameters-schema JSON { customerId:string, required }. Add DB select: select all fields listed from CUSTOMER where CustomerID=:customerId. Use attributes.queryParams.customerId for the input param. Add DataWeave to transform the first row into a structured JSON as per CUSTOMER schema (customerId, firstName, lastName, email, phoneNumber, dateOfBirth formatted, address object, createdAt/updatedAt ISO). Add a correct <mcp:responses> block for the MCP connector version in use.

### Prompt D (Properties)
In src/main/resources/config.properties, define db.host, db.port, db.user, db.systemPassword, db.serviceId, and MCP props: mcp.clientName, mcp.clientVersion, mcp.serverUrl.

### Prompt E (Build)
Run mvn clean package -DskipTests and fix any schema validation errors, especially on <mcp:responses>, per connector docs.

### Prompt F (Deploy)
Deploy to CloudHub 2.0 via mule-maven-plugin with cloudhub2=true; set region, workers, workerType, app.name; pass appProperties and secureProperties (db.systemPassword). After RUNNING, use the public CloudHub URL as mcp.serverUrl in config.


