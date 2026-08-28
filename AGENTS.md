# AGENTS.md

`claude-any-llm` is a single POSIX `sh` script (no package manager, no test
runner, no build). This repo is the **published mirror**; the canonical script
is `llm-gateway/scripts/claude-any-llm` in `llm-inference-network`, and the
launcher's behavioral tests live beside it in
`llm-gateway/backend/tests/unit/test_scripts/`. Edit the canonical script and
republish with `llm-gateway/scripts/publish-claude-any-llm.sh`; do not edit
`claude-any-llm` here directly.

## Test and verification doctrine

**1. Observable-boundary mandate, with an honest zero.** A test asserts
behavior at an observable boundary — this launcher's boundaries are exit
status, stderr/stdout text, the environment and argv it hands to `claude`, and
the files it writes (`~/.config/claude-llm/*.model`, `~/.claude/settings.json`).
Never assert on internals, and never assert on the text of the artifact under
test. If a change has no reachable boundary, write NO test. A manufactured
assertion is worse than none: it costs maintenance, blocks refactors, and
reports green for behavior nobody proved.

**2. Tautological-test ban.** A test that reads the source or config file whose
behavior it claims to prove, and asserts on that file's text, is banned. It
restates the implementation instead of exercising it, so it passes for a broken
script and fails for a correct rewrite. The only carve-out is a generated-output
gate, where the text *is* the contract — here that is the mirror drift check
(`cmp` against canonical), which legitimately compares bytes because byte
equality is the published guarantee. Deciding question before committing any
test: **would it fail under a plausible regression implemented with different
text?** If no, delete it.

**3. Probes are not tests.** An agent self-check — running the script to see
what it does, or asserting the inverse of a mistake the agent just made — is a
probe. Probes are ephemeral: run them, read the output, delete them before
commit. Do not promote a probe into the suite to prove you fixed something;
that pins your own error's shape, not a contract anyone depends on. Committed
tests pin durable behavioral contracts only.

**4. Cheapest lane first.** Climb only as far as the change requires; stop at
the first lane that can actually catch the regression.

| Lane | Command | Proves |
| --- | --- | --- |
| 1. Parse | `sh -n claude-any-llm` | The script is valid POSIX `sh`. |
| 2. Lint | `shellcheck -s sh -e SC1090,SC2016,SC1087 claude-any-llm` | No new shell defects. |
| 3. Drift | `cmp claude-any-llm ~/Developer/git/llm-inference-network/llm-gateway/scripts/claude-any-llm` | The mirror is byte-identical to canonical, so canonical's suite covers this file. |
| 4. Probe | `./claude-any-llm --model` (and other argv errors) | Pre-network argument handling, without credentials. Ephemeral — never commit. |
| 5. Behavior | in `llm-inference-network/llm-gateway/backend`: `uv run --quiet python -m pytest -q tests/unit/test_scripts/test_claude_any_llm_launcher.py tests/unit/test_scripts/test_claude_any_llm_model_lock.py` | The launcher's committed contracts. New tests go here, never in this repo. |

The three lane-2 exclusions are verified non-defects, not silencing: SC1087 is a
false positive (`$reserved_header[[:space:]]` is regex text in POSIX `sh`, which
has no arrays), SC1090 is a deliberately runtime-variable `. "$env_file"`, and
SC2016 is the intentionally single-quoted `sh -c` body whose expansion must be
deferred to the re-exec. Drop an exclusion the moment its line changes.

Lanes 1-4 need no credentials and no network. Anything past argument parsing
(catalog fetch, Doppler credential resolution, launching `claude`) requires the
gateway and a key, so it is not a local lane — it belongs to lane 5.
