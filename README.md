# pi-doorman

**An agent that carries API keys in its environment will leak them: into a command line you can
see in the transcript, into a log file, into a pull request body. The only key an agent cannot
leak is one it never has.**

[doorman](https://github.com/kmatzen/doorman) is a local HTTP proxy that holds credentials. The
agent refers to a credential by name; the proxy attaches the real key on its way to the one host
and method the credential is scoped to. The agent never sees the key, and a 403 tells it plainly
when a request falls outside that scope.

pi-doorman is the pi-side companion. It does two things, and both matter:

1. It registers the doorman MCP server, giving the agent `doorman_credentials` (list credential
   names with their allowed hosts and methods) and `doorman_request` (call a keyed service through
   the proxy, correctly, in one tool call).
2. It appends a short section to the system prompt with the usage rules that make the tools
   reliable: reach keyed services through the named credential, never set an Authorization header
   yourself, use `http://` through the proxy, and read a 403 as the allowlist doing its job.

The second half is not decoration. A tool a model has never been told about is a tool it uses
inconsistently: it will try the bare `curl`, get a 407, retry with the key pasted into the URL,
and undo the whole design in one chain of reasonable moves. The guidance is what makes the
consistency boring. It also says what doorman is *not* — a network filter. Unkeyed traffic takes
the normal path; the proxy exists to scope keys, not to police the internet.

The security property lives in doorman, not in pi. An agent that ignores every word of the
guidance still cannot extract a key it was never given; that is the point of the split, and the
reason this companion ships guidance and zero secrets.

## Setup

You need doorman itself ([kmatzen/doorman](https://github.com/kmatzen/doorman)); this extension
is the agent-side half. It locates the doorman MCP server in this order:

1. the `DOORMAN_MCP` environment variable,
2. `mcp/doorman-mcp.mjs` next to the installed package,
3. `~/git/doorman/share/mcp/doorman-mcp.mjs`.

`DOORMAN_PROXY` is passed through if set, for a proxy on a non-default address.

## Install

```bash
pi install npm:pi-doorman
```

TypeScript source loaded by pi directly; `@earendil-works/pi-coding-agent` is a peer dependency
and never bundled.

## License

MIT
