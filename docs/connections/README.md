# Connections and integrations

Connections give agents access to external systems without making the ReAct runtime depend on any one vendor.

The model is:

```text
integration
  -> connection
  -> capabilities
  -> tools
  -> agent bindings
```

An integration describes a kind of service. A connection is an authenticated instance of that service. Capabilities are the operations discovered or configured for a connection. Tools are the normalized functions that agents can call.

## Initial scope

The first integrations are:

1. **Web** — generic search, page retrieval, and browser/UI interaction where a normal HTTP API is not available.
2. **Slack** — workspace search, thread inspection, and message actions.

Microsoft Teams, GitHub, Notion, Google Drive, Linear, Jira, Salesforce, and MCP servers should use the same model later.

## Shared connection contract

```ts
type Connection = {
  id: string;
  organizationId: string;
  integrationId: string;
  name: string;
  scope: "organization" | "team" | "user";
  status: "pending" | "connected" | "expired" | "revoked" | "error";
  credentialRef?: string;
  metadata: Record<string, unknown>;
  createdBy: string;
  createdAt: string;
  lastCheckedAt?: string;
};
```

Credentials are referenced through a secret manager. Access tokens, API keys, cookies, and refresh tokens must not be stored directly in the application database or run events.

Every exposed capability must have:

- a stable internal tool name;
- an input schema;
- an output shape;
- a timeout;
- a policy and approval classification;
- a connection reference;
- an audit event for each invocation.

## Tool resolution

The ReAct runtime should resolve every tool call through the same path:

```text
model decision
  -> tool registry
  -> agent binding check
  -> policy check
  -> approval check
  -> connection lookup
  -> credential resolution
  -> integration adapter
  -> normalized result
```

The model never receives credentials or raw connection configuration. It only sees the name, description, and input schema of tools available to the current agent.

## Connection lifecycle

```text
not configured
  -> pending authorization
  -> connected
  -> capability discovery
  -> active
  -> expired / revoked / disabled
```

Connection checks should be safe and read-only. A failed health check must disable the connection or report an actionable error rather than silently retrying side-effecting operations.

## Design principles

- Integrations are adapters, not special cases in the agent loop.
- Read operations should be separated from side-effecting operations.
- Side-effecting tools should support approval and narrow input restrictions.
- Agent versions should pin the connection and capability configuration they use.
- Discovered capabilities should be reviewable before they become agent tools.
- Tool results should be normalized but should retain source metadata for auditability.
