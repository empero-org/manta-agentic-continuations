# 0.9M tokens of context

899,848 tokens, 3,337 earlier messages of one real coding-agent session. The session is cut right after a tool result, and Manta writes the next assistant turn on its own. One sample, unedited. The full text of every turn is in [`session1-0.9M.json`](session1-0.9M.json).

The session was logged from a current frontier agent. By this cut, the frontier agent had been compacted once; Manta has all 899,848 tokens in context.

## The last 4 turns before the cut

*Thinking*

> The retrain job for identity3 should have completed by now given it was only about 38 steps, so I'm checking whether the sft_final output exists and running the re-test manually to diagnose what's happening.

Re-test output's empty — retrain or re-test still going. Let me check directly:

**Tool call: `Bash`** (Check identity3 retrain done + re-test running state)

````
ssh -p [REDACTED] -i [REDACTED]/.ssh/id_ed25519 root@[REDACTED] 'cd /root/[REDACTED] && echo "identity3 final: $([ -f train/runs/[REDACTED]/sft_final.pt ] && echo YES || echo no)"; echo "train tail:"; grep -E "saved|val_loss" identity3_train.log 2>/dev/null | tail -2; echo "python procs: $(pgrep -fc "[p]ython")"; echo "gpu: $(nvidia-smi --query-gpu=utilization.gpu,memory.used --format=csv,noheader)"' 2>/dev/null
````

**Tool result**

````
identity3 final: YES
train tail:
VAL {'step': 38, 'epoch': 2.959, 'val_loss': 1.3226, 'best': 1.3226}
saved /root/[REDACTED]/train/runs/[REDACTED]/sft_final.pt (best val 1.3226)
python procs: 16
gpu: 10 %, 908 MiB
````

## Manta takes over

*Thinking*

> The identity3 training completed with a better validation loss than identity2, but the GPU is idle which means the re-test either finished or crashed. I'm trying to read the final file to check the status.

`identity3` is **done** (val 1.32) — GPU's idle, so the re-test should be finished too. Reading it:

> **What happened next.** The two read the same GPU check (908 MiB in use) differently: the frontier agent says the re-test is "running on the GPU right now"; Manta reads the GPU as idle and the re-test as finished. It was finished: its results came back seconds later.

## What the frontier agent wrote next

*Thinking*

> The identity3 retrain completed successfully with a validation loss of 1.3226, which is a slight improvement over identity2. The re-test is now running with 16 processes handling the evaluation and tokenizer workers, using about 10% GPU and 908MiB of memory. I'm waiting for the watcher to report back on the results.

`identity3` finished — and a good sign: **val 1.3226**, better than identity2's 1.34 and near the pre-persona sft_best (1.295), so the gentler pass degraded general ability much less. The re-test (task-following + identity) is running on the GPU right now (that's the 16 procs). I'll have the verdict — does she write hello-world cleanly *and* still know she's [REDACTED] — in a minute or two when the watcher fires. Holding for it.
