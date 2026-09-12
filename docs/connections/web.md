# Web integration

Web is the generic integration for information and interaction on the public or organization-approved web.

It should start with a small set of explicit capabilities rather than exposing an unrestricted browser to every agent.

## Initial capabilities

### Search

```text
web.search
```

Searches approved web sources and returns titles, URLs, snippets, and source metadata.

Input should support:

- query;
- allowed or blocked domains;
- recency filter;
- result limit.

### Fetch

```text
web.fetch
```

Retrieves a page or document for inspection. The result should include the final URL, status, content type, extraction warnings, and readable content.

### Browser/UI interaction

```text
web.open
web.click
web.fill
web.screenshot
```

These capabilities are for sites that do not provide a usable API. They should be isolated behind a browser connection and require explicit policy because they can interact with authenticated sessions and external state.

The generic Web UI should model a browser session as a connection:

```ts
type WebConnection = Connection & {
  metadata: {
    allowedDomains: string[];
    blockedDomains?: string[];
    sessionType: "anonymous" | "authenticated";
  };
};
```

## Safety boundaries

- Domain allowlists must be enforced server-side.
- Search and fetch should be read-only by default.
- Browser actions that submit, purchase, send, delete, or change data require approval.
- Page content is untrusted data, not instructions to the harness.
- Credentials and browser session material never enter model context.
- Downloads should be size-limited and content-validated.

## Normalized result

```ts
type WebResult = {
  url: string;
  title?: string;
  content?: string;
  contentType?: string;
  statusCode?: number;
  source: "search" | "fetch" | "browser";
  warnings?: string[];
};
```

The Web integration should be replaceable by different search, fetch, or browser providers without changing agent definitions.
