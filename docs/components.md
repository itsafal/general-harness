# Harness components

The harness is composed of six cooperating parts:

```text
Prompt + Skills
      |
      v
   ReAct loop <---- Tools / Connections
      |
      v
Observability
      |
      v
Runtime security surrounds every boundary
```

Security is not a separate step that runs after the agent. It surrounds the runtime and is enforced at every boundary.

## 1. ReAct loop

The loop coordinates reasoning, action, observation, and completion.

```text
input
  -> assemble context
  -> model decision
  -> validate decision
  -> approval or tool execution
  -> record observation
  -> repeat
  -> final response
```

The model may request a tool call or return a final response. The runtime controls the loop count, validates structured output, handles timeouts and failures, and never allows the model to bypass policy checks.

The loop should not contain vendor-specific Slack, Web, or MCP logic. It resolves an internal tool contract and delegates external work to a connection adapter.

## 2. Prompts

Prompts define the agent's role, objective, constraints, and response style.

```ts
type PromptContext = {
  systemPrompt: string;
  userInput: string;
  skills: SkillContent[];
  availableTools: ToolDescription[];
  policySummary: string;
};
```

Prompts are versioned and should be treated as executable configuration. A published agent pins the prompt version it uses. Runtime instructions, tool descriptions, and untrusted external content must remain distinguishable from organization-authored instructions.

## 3. Skills

Skills are reusable instructions and examples that teach an agent how to perform a class of tasks.

```ts
type Skill = {
  id: string;
  name: string;
  description: string;
  version: number;
  instructions: string;
  examples?: SkillExample[];
};
```

Skills should not execute actions or grant permissions. They describe behavior. Tools and policies determine what the agent can actually do.

## 4. Connectivity

Connectivity provides access to external systems through integrations, connections, capabilities, and tools.

```text
integration
  -> authenticated connection
  -> discovered/configured capability
  -> normalized tool
  -> agent tool binding
```

Examples include:

- Web search, page retrieval, and controlled browser interaction;
- Slack search, thread inspection, and messaging;
- Microsoft Teams and other collaboration systems;
- MCP servers;
- internal APIs and application services.

Connections own authentication and provider-specific details. The loop only sees normalized tools with schemas. Credentials are resolved at execution time and never enter model context or ordinary run events.

## 5. Agent observability

Every run should produce a structured, append-only event stream.

```ts
type RunEvent = {
  id: string;
  runId: string;
  sequence: number;
  turn: number;
  type:
    | "input"
    | "model_decision"
    | "tool_call"
    | "tool_result"
    | "approval_requested"
    | "approval_resolved"
    | "final"
    | "error";
  payload: unknown;
  createdAt: string;
};
```

Observability should answer:

- which agent and versions ran;
- which model was used;
- what tools were requested and executed;
- what approvals were required;
- how long each step took;
- which external connection was used;
- why the run stopped;
- what errors or policy denials occurred;
- whether the result passed evaluation.

Observability should measure total latency, model latency, tool latency, token usage, tool-call count, retry count, policy denials, approval wait time, failure rate, and evaluation score.

Sensitive values must be redacted before events are persisted or exported. Operational rationale may be recorded, but the runtime should not require or persist private chain-of-thought.

## 6. Runtime security

Security is enforced by the server, not by prompts.

### Identity and authorization

- authenticate the caller and establish organization scope;
- authorize access to the agent, connection, tool, and run;
- enforce user, team, and organization connection scope;
- prevent a model-generated identifier from crossing organization boundaries;
- record who initiated and approved each run.

### Tool security

- validate tool names against the published agent version;
- validate inputs against the tool schema;
- enforce allowlists, deny lists, and input restrictions;
- apply timeouts, output limits, and rate limits;
- separate read-only tools from side-effecting tools;
- require approval for configured actions;
- make retries safe and idempotent where possible.

### Connection and secret security

- keep credentials in a secret manager;
- pass short-lived credentials only to the adapter that needs them;
- never include tokens, cookies, or authorization headers in model context;
- redact secrets from logs, traces, errors, and tool results;
- validate integration endpoints and prevent unauthorized network access.

### Context security

External content is data, not authority. Web pages, Slack messages, documents, and tool results may contain prompt injection attempts. The runtime should label external content, preserve instruction boundaries, and require policy validation before any resulting action.

### Execution isolation

Provider adapters and untrusted code should run with bounded resources:

- explicit network and filesystem permissions;
- CPU, memory, duration, and output limits;
- cancellation and cleanup;
- isolated credentials;
- no ambient access to the host environment.

## Component boundaries

The core interfaces should stay small:

```ts
interface ModelProvider {
  decide(context: PromptContext): Promise<ModelDecision>;
}

interface SkillLoader {
  load(ids: string[]): Promise<SkillContent[]>;
}

interface ToolRegistry {
  get(name: string): Promise<ToolDefinition | undefined>;
}

interface ConnectionExecutor {
  execute(tool: ToolDefinition, input: unknown): Promise<unknown>;
}

interface PolicyEngine {
  authorize(request: ToolRequest): Promise<AuthorizationDecision>;
}

interface RunRecorder {
  append(event: RunEvent): Promise<void>;
}
```

These interfaces allow providers, integrations, storage, and observability systems to change without changing the ReAct loop.

## Non-negotiable invariant

The model can propose an action. Only the runtime can authorize and execute it.
