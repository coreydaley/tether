# Tether

A durable, sandboxed runner for AI agent jobs, with an eval suite that publishes its pass rates.

> **Status: early.** The proof of concept is being built. Nothing here is ready to use yet.

## What it is

An AI agent works in a loop: the model picks a step, a tool runs, the result
goes back to the model, and it repeats until the task is done. Tether owns
that loop and adds the three things you need before you can leave an agent
running on its own:

- **It survives a crash.** Every step is written to a log before it is acted
  on. Kill the runner halfway through a job, start it again, and the job
  resumes from the last recorded step without running a finished tool call
  twice.
- **It stays in its box.** Tools run inside a sandbox, never on the host,
  with credentials that last only as long as the job.
- **It is measured.** Tether ships with its own eval suite. The pass rates
  are committed to this repo as they are, including the ones that look bad.

## Models

Model providers are set in a JSON config file. Tether talks to any
OpenAI-compatible endpoint, so you can develop against a local model server
such as [Ollama](https://ollama.com) for free and run benchmarks against
hosted models when you want real numbers. API keys live in a local `.env`
file that is never committed.

## Roadmap

1. **Proof of concept.** The agent loop, the step log, kill-and-resume, two
   tools running in a container, and two evals (task completion and crash
   recovery).
2. **Hardening.** gVisor as the container runtime, an egress allowlist, and
   short-lived per-job credentials.
3. **Security evals and benchmarks.** Prompt injection in tool output,
   blocked exfiltration, and a comparison across local and hosted models.

Work is tracked in [issues](../../issues).

## Docs

- [Overview](docs/Overview.md): scope, known hard parts and open questions.
- [Changelog](CHANGELOG.md): what changed and why.
- [Agent instructions](AGENTS.md): how AI coding agents work in this repo.

## Contributing

This is an early, single-maintainer project. Issues and discussion are
welcome; please open an issue before sending a pull request.

## License

[Apache-2.0](LICENSE)
