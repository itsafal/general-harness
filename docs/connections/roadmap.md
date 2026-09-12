# Integration roadmap

All future integrations should implement the shared connection and tool contracts.

| Integration | Initial purpose | Likely auth | Priority |
| --- | --- | --- | --- |
| Web | Search, fetch, browser/UI tasks | Provider or browser session | First |
| Slack | Workspace search and messaging | OAuth2 | First |
| Microsoft Teams | Teams, channels, threads, messages | OAuth2 | Next |
| GitHub | Issues, pull requests, repository context | OAuth2 or app token | Next |
| Google Drive | Search and retrieve documents | OAuth2 | Next |
| Notion | Search and update pages | OAuth2 or API key | Later |
| Linear / Jira | Issues and project workflows | OAuth2 or API token | Later |
| Salesforce | Customer and account context | OAuth2 | Later |
| MCP | Discover tools from an MCP server | Server-specific | Later |

The roadmap is intentionally capability-driven. A new vendor should be added when it solves a real organizational use case, not because the harness needs a catalog of every possible connector.
