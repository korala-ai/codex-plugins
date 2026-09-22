# Korala for Codex

This directory is the root of Korala's Codex plugin marketplace. Publish it as
a public repository, for example `korala-ai/codex-plugins` (the contents of
this directory become the repository root). The ChatGPT desktop app reads the
same plugin format.

## Install

```
codex plugin marketplace add korala-ai/codex-plugins
codex plugin add korala@korala
codex mcp login korala
```

The last command opens a browser. Sign in to Korala and pick what Codex may
do on the consent page. Sending is off unless you tick it.

The plugin contains:

- `.codex-plugin/plugin.json`: the manifest and the directory listing text.
- `.mcp.json`: the hosted Korala MCP server (`https://api.korala.ai/mcp`,
  OAuth sign-in, no keys) with `required: true` and
  `startup_timeout_sec: 60`.
- `skills/korala-documents`: how to draft, prepare, send and track with those
  tools, and the no-account draft link for people who have not connected.
- `assets/korala.svg`: the icon Codex shows next to the plugin.

## Why `required: true`

Codex builds each turn's tool list from the MCP servers that are ready. It
waits one second for an optional server (`mcp_optional_startup_grace_ms`,
default 1000) and leaves out any server still connecting. Korala's hosted
server takes about two seconds to connect, so without `required` its tools
appear in some turns and are missing in others. `required: true` makes Codex
wait for the server, and `startup_timeout_sec: 60` gives the OAuth handshake
room on a slow network. The trade-off: `codex exec` stops with an error when
Korala cannot connect, for example before you have signed in.

## Try it locally

Use a throwaway `CODEX_HOME` so your own configuration stays untouched:

```
export CODEX_HOME="$(mktemp -d)"
codex plugin marketplace add ./integrations/codex-plugin
codex plugin add korala@korala
codex mcp list
```

`codex mcp list` shows `korala` with `OAuth` auth.

## Publishing to the plugin directory

OpenAI lists public plugins in one directory shared by ChatGPT and Codex,
through its plugin submission portal. We have not submitted this plugin. The
ChatGPT app listing (see `integrations/directory-listings`) covers the same
server.

## Keeping the helper in sync

`plugins/korala/skills/korala-documents/scripts/korala.mjs` is a copy of
`integrations/agent-skills/korala-documents/scripts/korala.mjs`.
`node scripts/package-integrations.mjs --check` fails when they differ.

Sources checked on 2026-09-22:

- https://developers.openai.com/codex/plugins/build
- `codex-rs/codex-mcp/src/plugin_config.rs` and
  `codex-rs/config/src/mcp_types.rs` at tag `rust-v0.155.1` of
  https://github.com/openai/codex (the `.mcp.json` fields Codex accepts)
