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
curl -fsSL -o ~/.local/bin/claude-any-llm \
  https://raw.githubusercontent.com/williamacallahan/claude-any-llm/main/claude-any-llm
chmod +x ~/.local/bin/claude-any-llm
```

The default model is `qwen3.8-flash-long-context` through the development Squirrel gateway. The launcher:

- reads the live gateway compatibility catalog instead of restating model metadata;
- applies the published context and output limits to Claude Code;
- shows the selected model's pricing, reasoning levels, and capabilities;
- accepts `-m`/`--model` to select another gateway model;
- runs Claude Code with its own `CLAUDE_CONFIG_DIR`, `~/.config/claude-llm/claude-any-llm/`, whose transcripts, history, `.claude.json`, plans, skills, plugins, and peer session registry link back to the shared `~/.claude`, so `/resume` and history stay shared with plain `claude`;
- rebuilds that directory's private `settings.json` each launch from the shared settings minus `model` and `effortLevel`, so an in-session `/model` or `/effort` pick persists there and is reused on the next launch, and never edits the shared `~/.claude/settings.json`; and
- prints the authoritative gateway session cost, request count, token totals, and cache-read portion on exit, including Ctrl-C exits.

Credentials are resolved in this order:

1. `ANTHROPIC_AUTH_TOKEN` or `ANTHROPIC_API_KEY` already in the environment;
2. `~/.config/claude-llm/claude-any-llm.env` (or `CLAUDE_LLM_ENV_FILE`);
3. `LLM_GATEWAY_DEV_API_KEY` from Doppler `personal/dev`.

Examples:

```sh
claude-any-llm
claude-any-llm --model stealth/ox-alpha
claude-any-llm --model qwen3.6-plus --verbose
```

Claude Code's `/usage` remains a client estimate for custom models. The `Gateway actual:` line printed after the session is the gateway's persisted provider-resolved ledger.
