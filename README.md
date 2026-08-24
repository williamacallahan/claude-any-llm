# claude-any-llm

Gateway-backed `claude-any-llm` launcher: run Claude Code against any model the
development Squirrel gateway publishes in its compatibility catalog.

Canonical source: `llm-inference-network/llm-gateway/scripts/claude-any-llm`
(tests live beside it in `llm-gateway/backend/tests/unit/test_scripts/`). This
repo is the published copy; sync it with
`llm-gateway/scripts/publish-claude-any-llm.sh` from that repo — do not edit
the script here directly.

## Install

```sh
gh api repos/williamacallahan/claude-any-llm/contents/claude-any-llm \
  -H "Accept: application/vnd.github.raw" > ~/.local/bin/claude-any-llm
chmod +x ~/.local/bin/claude-any-llm
```

The default model is `stealth/ox-alpha` through the development Squirrel gateway. The launcher:

- reads the live gateway compatibility catalog instead of restating model metadata;
- applies the published context and output limits to Claude Code;
- shows the selected model's pricing, reasoning levels, and capabilities;
- accepts `-m`/`--model` to select another gateway model;
- remembers an in-session `/model` pick in `~/.config/claude-llm/claude-any-llm.model` and reuses it on the next launch;
- scrubs gateway catalog ids out of `~/.claude/settings.json` before launch and on exit, restoring the prior default so plain `claude` sessions against api.anthropic.com keep working; and
- prints the authoritative gateway session cost, request count, token totals, and cache-read portion on exit, including Ctrl-C exits.

Credentials are resolved in this order:

1. `ANTHROPIC_AUTH_TOKEN` or `ANTHROPIC_API_KEY` already in the environment;
2. `~/.config/claude-llm/claude-any-llm.env` (or `CLAUDE_LLM_ENV_FILE`);
3. `LLM_GATEWAY_DEV_API_KEY` from Doppler `personal/dev`.

Examples:

```sh
claude-any-llm
claude-any-llm --model stealth/ox-alpha
claude-any-llm --model qwen3.8-27b --verbose
```

Claude Code's `/usage` remains a client estimate for custom models. The `Gateway actual:` line printed after the session is the gateway's persisted provider-resolved ledger.
