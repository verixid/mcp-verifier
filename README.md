# VerixID MCP — Registry & Usage Guide

## 1. Overview

VerixID has a production MCP server registered in the Official MCP Registry.

- MCP server: `https://mcp.verixid.com/`
- Registry name: `com.verixid/verifier`
- Version: `1.0.0`
- Transport: Streamable HTTP
- Authentication: Not required for MCP server access
- Official Registry status: `active`
- Server Card: `https://mcp.verixid.com/.well-known/mcp.json`

The MCP server provides the `verixid_verify` tool for verifying VerixID proof-of-existence records.

## 2. Production Architecture

```text
AI / MCP Client
      |
      v
Official MCP Registry
      |
      | discover
      v
com.verixid/verifier
      |
      v
https://mcp.verixid.com/
      |
      v
VerixID Verifier
      |
      +-- verixid_verify
```

The Registry contains discovery metadata only. The actual MCP service remains hosted at `mcp.verixid.com`.

## 3. MCP Server Card

Discovery URL:

```text
https://mcp.verixid.com/.well-known/mcp.json
```

Current server card:

```json
{
  "$schema": "https://static.modelcontextprotocol.io/schemas/mcp-server-card/v1.json",
  "version": "1.0",
  "protocolVersion": "2025-11-25",
  "serverInfo": {
    "name": "verixid-verifier",
    "title": "VerixID Verifier",
    "version": "1.0.0"
  },
  "transport": {
    "type": "streamable-http",
    "endpoint": "/"
  },
  "authentication": {
    "required": false
  }
}
```

## 4. Official Registry Entry

Registry identifier:

```text
com.verixid/verifier
```

Production endpoint:

```text
https://mcp.verixid.com/
```

Registry status:

```text
active
isLatest: true
version: 1.0.0
```

Registry search:

```bash
curl -s   "https://registry.modelcontextprotocol.io/v0.1/servers?search=com.verixid/verifier"   | jq
```

Expected result contains:

```text
name: com.verixid/verifier
status: active
version: 1.0.0
```

## 5. Tool

The MCP server exposes:

```text
verixid_verify
```

Purpose:

Verify a VerixID proof-of-existence record using a Record ID and, when available, an Ownership Key.

Conceptually:

```text
Record ID
    |
    v
VerixID verification
    |
    +-- record exists
    +-- hash/proof verification
    +-- ownership verification when Ownership Key is supplied
```

The server does not require or process the original file.

## 6. How to Use from an MCP Client

The recommended discovery identifier is:

```text
com.verixid/verifier
```

If the MCP client supports registry search:

1. Search for `com.verixid/verifier`.
2. Select `VerixID Verifier`.
3. Connect to the registered remote server.
4. The client should discover the `verixid_verify` tool.
5. Call the tool with the VerixID Record ID.
6. Supply the Ownership Key when ownership verification is required.

The direct MCP endpoint is:

```text
https://mcp.verixid.com/
```

## 7. Direct Server Discovery

The server card can be checked with:

```bash
curl -s https://mcp.verixid.com/.well-known/mcp.json | jq
```

Expected key properties:

```text
serverInfo.name       = verixid-verifier
serverInfo.title      = VerixID Verifier
serverInfo.version    = 1.0.0
protocolVersion       = 2025-11-25
transport.type        = streamable-http
transport.endpoint    = /
authentication.required = false
```

## 8. Transport Behavior

A normal GET request to the MCP endpoint can return:

```text
HTTP 406
Not Acceptable: Client must accept text/event-stream
```

This is expected when the client does not provide an acceptable streaming transport header.

For example:

```bash
curl -i   -H "Accept: text/event-stream"   https://mcp.verixid.com/
```

The server responds with:

```text
HTTP/2 200
content-type: text/event-stream
```

and keeps the connection open.

This is not an application timeout. The MCP transport is waiting for the client protocol interaction.

## 9. Registry Manifest

The registry manifest used for publishing is:

```json
{
  "$schema": "https://static.modelcontextprotocol.io/schemas/2025-12-11/server.schema.json",
  "name": "com.verixid/verifier",
  "title": "VerixID Verifier",
  "description": "Verify VerixID proof-of-existence records and optionally prove ownership.",
  "version": "1.0.0",
  "websiteUrl": "https://verixid.com",
  "remotes": [
    {
      "type": "streamable-http",
      "url": "https://mcp.verixid.com/"
    }
  ]
}
```

Recommended local filename:

```text
server.json
```

## 10. Registry Publisher Setup

The registry publisher binary used:

```text
mcp-publisher
```

Available commands:

```text
init
login
logout
publish
status
validate
```

Validate without publishing:

```bash
./mcp-publisher validate
```

Publish:

```bash
./mcp-publisher publish
```

## 11. DNS Authentication

The namespace `com.verixid` was authenticated using DNS ownership of:

```text
verixid.com
```

The publisher supports:

```text
mcp-publisher login dns
```

The authentication flow uses an Ed25519 key pair.

Example:

```bash
./mcp-publisher login dns   --domain verixid.com   --private-key "$PRIVATE_KEY_HEX"
```

The Registry provides an expected DNS proof record. The public key is placed in the DNS TXT record for `verixid.com`.

Important:

- Never commit the Ed25519 private key.
- Never publish the private key in documentation.
- Never send the private key through chat or email.
- Keep the private key in a protected local/secret-management location.
- If a private key is exposed, revoke its use and generate a replacement.

## 12. Key Storage

The local registry project used:

```text
verixid-mcp-registry/
```

Example structure:

```text
verixid-mcp-registry/
├── server.json
├── mcp-publisher
├── registry-ed25519.pem
└── .gitignore
```

The private key should be excluded:

```text
registry-ed25519.pem
```

Recommended `.gitignore` entry:

```text
registry-ed25519.pem
```

## 13. Publishing Procedure

Future version publishing should follow this sequence:

```text
1. Update MCP server
2. Update server version
3. Update server card if required
4. Update server.json
5. Validate
6. Authenticate if required
7. Publish
8. Verify Registry entry
9. Test MCP discovery/tool execution
```

Validation:

```bash
./mcp-publisher validate
```

Publishing:

```bash
./mcp-publisher publish
```

Registry verification:

```bash
curl -s   "https://registry.modelcontextprotocol.io/v0.1/servers?search=com.verixid/verifier"   | jq
```

## 14. Current Production Status

As of the initial publication:

```text
Registry name:       com.verixid/verifier
Version:             1.0.0
Status:              active
Latest:              true
Transport:           streamable-http
Endpoint:            https://mcp.verixid.com/
Authentication:      not required
```

The server was successfully published to the Official MCP Registry.

## 15. Troubleshooting

### HTTP 406 from MCP endpoint

Example:

```text
HTTP 406
Not Acceptable: Client must accept text/event-stream
```

Check that the MCP client is using the correct MCP transport and `Accept` headers.

Do not treat this response alone as a server failure.

### SSE connection appears to hang

A command such as:

```bash
curl -N --max-time 5   -H "Accept: text/event-stream"   https://mcp.verixid.com/
```

may end with:

```text
Operation timed out
```

with no received body.

This can be normal because the server keeps the streaming connection open and waits for MCP protocol interaction.

### Registry validation fails

Run:

```bash
./mcp-publisher validate
```

The Registry enforces field constraints. For example, the server description must not exceed the Registry's allowed length.

### DNS authentication fails

Typical error:

```text
no MCP public key found in DNS TXT records
```

Check:

```bash
dig TXT verixid.com +short
```

Ensure the Registry-provided MCP TXT proof is present and has propagated.

## 16. Security Notes

The MCP server is intentionally zero-custody:

- It does not require the original file for verification.
- Verification is based on VerixID record data.
- Ownership verification can use the Ownership Key.
- MCP server authentication is currently not required.

Registry authentication is separate from MCP server authentication.

The Ed25519 key used for Registry namespace authentication must be treated as a signing credential and must never be included in public repositories or documentation.

## 17. Useful URLs

VerixID:

```text
https://verixid.com
```

MCP endpoint:

```text
https://mcp.verixid.com/
```

MCP Server Card:

```text
https://mcp.verixid.com/.well-known/mcp.json
```

MCP documentation:

```text
https://verixid.com/docs/mcp/
```

Official MCP Registry:

```text
https://registry.modelcontextprotocol.io/
```

Registry search:

```text
https://registry.modelcontextprotocol.io/?q=com.verixid%2Fverifier
```

## 18. Summary

VerixID is registered as an official remote MCP server:

```text
com.verixid/verifier
```

The production MCP service remains:

```text
https://mcp.verixid.com/
```

The Registry provides discovery; the VerixID MCP server provides the actual verification capability.

Current status:

```text
MCP server              ✓
Streamable HTTP         ✓
Server Card             ✓
Registry manifest       ✓
Registry validation     ✓
DNS namespace auth      ✓
Official publication    ✓
Registry status         active
```
