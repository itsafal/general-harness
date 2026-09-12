# Motivation

Organizations want to build useful AI agents without adopting a large orchestration framework or creating a separate platform for every team.

The goal of this project is to provide a small, generic harness for running ReAct-style agents. Teams should be able to create and adapt agents by changing configuration, prompts, skills, tools, policies, and integrations—not by learning ADK, LangChain, LangGraph, or another framework-specific programming model.

## The problem

Agent behavior usually combines a few predictable parts:

- instructions that define how the agent should behave;
- skills that provide reusable operating knowledge;
- tools that let the agent inspect or change external systems;
- policies that control permissions, approvals, and limits;
- integrations and connections such as Slack, MCP servers, and internal services;
- evaluations that show whether the behavior is reliable.

These parts should be easy for different teams to own and change independently. The runtime should provide the common execution, validation, and observability layer instead of embedding organization-specific behavior in code.

## The approach

The harness uses a simple ReAct loop:

```text
request
  -> observe context
  -> decide whether to answer or call a tool
  -> validate the decision
  -> execute the tool when allowed
  -> record the result
  -> repeat until a final answer
```

The model decides what to do. The harness validates that decision. Tools perform the action. Policies determine whether the action is allowed. Every meaningful step is recorded so a run can be inspected and evaluated later.

The runtime should not depend on a workflow graph, multi-agent protocol, or framework-specific abstraction. A workflow can start as a prompt, a set of skills, a set of tools, and a policy. More structure should be added only when repeated use demonstrates that it is necessary.

## What the organization owns

The organization should own the parts that define an agent's behavior:

- prompts and prompt versions;
- reusable skills and skill versions;
- tools and tool schemas;
- integrations and authenticated connections;
- policies and approval requirements;
- evaluation cases and expected behavior.

The harness owns the stable mechanics:

- loading an agent configuration;
- assembling model context;
- running the ReAct loop;
- validating structured model decisions;
- resolving tools and connections;
- enforcing policy;
- handling approvals, timeouts, retries, and cancellation;
- recording run events;
- running evaluations.

## Integrations are first-class

External systems should not be hard-coded into the agent loop. An integration describes a service such as Slack or MCP. A connection represents a specific authenticated workspace, account, or server. Capabilities discovered from a connection are exposed as normalized tools.

```text
integration
  -> connection
  -> capabilities
  -> tools
  -> agent bindings
```

This lets an agent use Slack, an MCP server, or an internal API through the same ReAct interface. Credentials remain outside the application database and are referenced through a secret manager.

## Design principles

1. **Small core.** The runtime should be understandable by a normal application team.
2. **Declarative customization.** Most new agents should require files and configuration, not new orchestration code.
3. **Explicit permissions.** The model may request an action, but the server decides whether it can happen.
4. **Immutable execution inputs.** Published agent versions are immutable, and every run records the exact versions it used.
5. **Inspectable behavior.** Tool calls, results, approvals, errors, and final responses are structured run events.
6. **Safe side effects.** Actions that change external state can require explicit approval and should be bounded by policy.
7. **Provider flexibility.** Model providers and integrations are adapters around the same internal contracts.
8. **Evidence over magic.** Evaluations and run traces should make reliability measurable.

## What this project deliberately avoids

This project is not intended to become:

- a general-purpose workflow engine;
- a visual graph builder;
- a collection of framework-specific agent abstractions;
- an opaque autonomous system with unrestricted tool access;
- a replacement for existing integration APIs;
- a place to store secrets directly.

The desired result is deliberately modest: a reliable, auditable execution harness that lets each team bring its own prompts, skills, tools, integrations, and policies.
