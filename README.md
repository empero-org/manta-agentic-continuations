# Manta agentic continuations

One real coding-agent session, logged from a current frontier agent, cut at a growing context size. Everything before the cut stays in context, up to 7.4M tokens. Each time, Manta writes the next assistant turn on its own: its reasoning and its message, up to the point where it runs its next tool.

**Read it on the web: [empero-org.github.io/manta-agentic-continuations](https://empero-org.github.io/manta-agentic-continuations/)**

Close to 1M tokens the frontier agent reached its limits: its context was compacted into a summary at 786,556 tokens, and 13 times by the last cut. Manta reads the whole session at every cut.

| Context | Earlier messages | Frontier agent compacted | Manta's next turn |
|---|---|---|---|
| [**0.9M**](continuations/session1-0.9M.md) | 3,337 | 1 | `identity3` is **done** (val 1.32) — GPU's idle, so the re-test should be finished too. Reading it: |
| [**1.5M**](continuations/session1-1.5M.md) | 5,550 | 2 | Let me see the remaining flags (model-reasoning-effort and sandbox): |
| [**3.5M**](continuations/session1-3.5M.md) | 12,919 | 6 | Model downloaded (19G). getdata hit the known `datasets-5.0` script block on `deepmind/pg19`. Fixing with a parquet-only source: |
| [**5.0M**](continuations/session1-5.0M.md) | 19,623 | 8 | Both waves are done, but I need to read this carefully — the log tail says **"all 2 waves complete"** and yet only wave 3 has episodes. Let me get … |
| [**7.4M**](continuations/session1-7.4M.md) | 31,450 | 13 | **DPO saved.** Three parallel actions: upload (protect the artifact), serve on 8080 for your tunnel, strict gate: |

## The test

- **The session:** one long, real coding-agent session: training runs on remote GPU boxes, data scripts, logs and fixes. It runs to about 7.5M tokens.
- **The frontier agent:** the session was run by a current frontier agent. Its context filled up and was compacted into a summary 13 times before the last cut, the first at 786,556 tokens and 12 of them automatically at about 1M tokens. Manta gets every cut whole, never compacted.
- **The cut:** the session is cut at a turn boundary right after a tool result, so the next turn is the agent's. Everything before the cut is the prompt.
- **The continuation:** one sample per cut, temperature 1.0, top_p 0.95, at most 2,048 new tokens. Nothing is edited or picked. Each sample ends with the agent's message, right before it runs its next tool.
- **Redactions:** IPs, ports, keys and tokens, emails, IDs and hashes, paths, and internal project and model names all read `[REDACTED]`. Nothing else is changed.

## Next to the frontier agent

Each file also has the turn the frontier agent actually wrote next.

| Context | Manta | The frontier agent |
|---|---|---|
| 0.9M | `identity3` is **done** (val 1.32) — GPU's idle, so the re-test should be finished too. Reading it: | `identity3` finished — and a good sign: **val 1.3226**, better than identity2's 1.34 and near the pre-persona … |
| 1.5M | Let me see the remaining flags (model-reasoning-effort and sandbox): | calls `Bash`: See remaining exec flags (sandbox, cd, git check, output) |
| 3.5M | Model downloaded (19G). getdata hit the known `datasets-5.0` script block on `deepmind/pg19`. Fixing with a … | Model is down (19 G). `getdata` failed — `deepmind/pg19` ships a loading script, which `datasets` v4 dropped. … |
| 5.0M | Both waves are done, but I need to read this carefully — the log tail says **"all 2 waves complete"** and yet … | calls `Bash`: Waves 3+4 training results |
| 7.4M | **DPO saved.** Three parallel actions: upload (protect the artifact), serve on 8080 for your tunnel, strict … | DPO composite saved. Executing the post-train sequence — HF upload and the vLLM install both launch now in … |

**At 0.9M:** The two read the same GPU check (908 MiB in use) differently: the frontier agent says the re-test is "running on the GPU right now"; Manta reads the GPU as idle and the re-test as finished. It was finished: its results came back seconds later.

## Files

```
index.html      all cuts on one page (the GitHub Pages site)
continuations/  one JSON and one Markdown file per cut
                session1-0.9M.json  session1-0.9M.md
                session1-1.5M.json  session1-1.5M.md
                session1-3.5M.json  session1-3.5M.md
                session1-5.0M.json  session1-5.0M.md
                session1-7.4M.json  session1-7.4M.md
```

Each JSON file has:

- `context_tokens` and `context_messages`: the size of the context.
- `context_tail`: the last 4 turns before the cut, in full.
- `continuation`: Manta's turn, as `thinking`, `text` and `tool_calls`.
- `original_next`: the turn the frontier agent wrote next.
- `original_agent`: who wrote the session, and how often its context had been compacted by this cut.
- `what_happened_next`: where the session settles a disagreement between the two turns.
- `sampling`: the settings above.

The Markdown file shows the same turns for reading.
