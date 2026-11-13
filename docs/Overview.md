# Tether

Tether runs AI agent jobs so that they survive a crash and can't escape their
box. An agent job is a loop: the model picks a step, a tool runs, the result
goes back to the model, and it repeats until the task is done. Tether owns that
loop. It writes every step to a log before acting on it, so a job that dies
halfway resumes from the last recorded step and never runs a finished tool call
twice. Tools run inside a sandbox, never on the host, with credentials that
last only as long as the job.

It is for engineers who want to run agents unattended and need to answer two
questions with numbers, not opinions: does the job finish correctly, and what
can it reach while it runs? So Tether ships with its own eval suite, and the
pass rates are published in this repo, including the ones that look bad.

The project is built in the open and run like client work: a PRD first, issues
for every change, a changelog, and docs in `docs/`. It starts as a small proof
of concept and grows by iteration.

## Scope

**Proof of concept**

- Agent loop owned by Tether: model call, tool call, result, repeat.
- Step log in SQLite, written before each step is acted on.
- Kill the runner mid-job, restart it, and the job resumes without repeating
  completed tool calls.
- Two tools (run a command, read and write a file) that execute in a
  container, never on the host.
- Model providers set in a JSON config file. The first provider type is
  OpenAI-compatible, which covers local servers such as Ollama. API keys live
  in a local `.env` file that is never committed.
- Two evals: one task-completion check and one crash-recovery check.

**After the proof of concept, in order**

1. gVisor as the container runtime, an egress allowlist, and short-lived
   per-job credentials.
2. Security evals (prompt injection in tool output, blocked exfiltration) and a
   benchmark across local and hosted models, with published results.

**Out for now**

- Multi-tenancy.
- A web UI.
- Kubernetes deployment.
- Wrapping existing agent CLIs.

## Known Hard Parts

- **Exactly-once tool effects on resume.** A crash between running a tool and
  recording its result is the case the step log has to get right.
- **gVisor needs Linux.** On macOS the sandbox has to run inside a Linux VM.
  This is untested on the development machines and is the first spike after
  the proof of concept.
- **Small local models are weak at tool calling.** Some will fail tasks for
  reasons unrelated to Tether. That is acceptable for testing the runner, and
  the comparison is worth publishing, but it has to be reported honestly.
- **Scoring nondeterministic runs.** The same task can pass on one run and
  fail on the next. Evals need deterministic checks and repeated runs.

## Open Questions

- Whether Anthropic's OpenAI-compatible endpoint is good enough to keep a
  single provider type, or whether a native Anthropic provider is needed.
- Which local models are usable for tool calling.
- Whether gVisor runs acceptably inside a Linux VM on Apple Silicon.
