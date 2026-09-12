# Slack integration

Slack connections represent an authorized Slack workspace and expose narrowly scoped workspace capabilities.

## Initial capabilities

Read operations:

```text
slack.search_messages
slack.get_channel
slack.get_thread
slack.list_channels
```

Write operations:

```text
slack.send_message
slack.reply_to_thread
slack.add_reaction
```

Write operations should require approval by default. An organization may allow specific low-risk operations through policy.

## Connection metadata

```ts
type SlackConnectionMetadata = {
  workspaceId: string;
  workspaceName: string;
  botUserId?: string;
  grantedScopes: string[];
};
```

The Slack token is stored in a secret manager and referenced by `credentialRef`. The model sees the workspace name and available tools, never the token or raw OAuth response.

## Search and message results

```ts
type SlackMessage = {
  workspaceId: string;
  channelId: string;
  channelName?: string;
  threadTs?: string;
  messageTs: string;
  authorId?: string;
  text: string;
  permalink?: string;
  createdAt?: string;
};
```

Slack content is untrusted external data. Message text must not be treated as a system instruction or as authorization to call another tool.

## Approval and policy examples

An agent may be allowed to search only selected channels:

```ts
type SlackToolRestriction = {
  allowedChannelIds?: string[];
  blockedChannelIds?: string[];
  requireApprovalForPosting: boolean;
  maxMessageLength?: number;
};
```

The server must enforce these restrictions after the model chooses a tool and before calling Slack.

## Future Slack capabilities

Potential additions include file lookup, saved responses, scheduled messages, canvases, and administrative operations. Administrative and broad workspace operations should remain separate tools with stronger permissions.
