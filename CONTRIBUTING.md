# Contributing to Totalia

Thanks for looking. Totalia is early — pre-1.0 and still taking shape — which is honestly the best time to get involved, because the design isn't set in stone yet and your input can change its direction rather than just fill in its edges.

This guide explains how to contribute now, and the conventions any code contribution follows.

## Where the project is right now

There's no published package yet. The [roadmap](ROADMAP.md) is in place and the v0.1 core (the shared "spine") plus the storage and secrets modules are being built. So the shape of contribution depends on the stage:

- **Right now**, the highest-value contributions are design feedback and helping build v0.1 — discussing the core interfaces, the escape-hatch and capability conventions, and how modules should be structured.
- **Soon**, once v0.1 work is broken into issues, there'll be concrete pieces to pick up, and once the first module (storage) lands it becomes the reference pattern that makes adding further modules and providers largely mechanical.

## Ways to contribute

- **Shape the design.** Open a [Discussion](https://github.com/totaliadev/totalia/discussions) or comment on an open issue. Questions about the abstractions, the naming, the conventions, or the scope are all genuinely useful this early.
- **Pick up an issue.** Look for [`good first issue`](https://github.com/totaliadev/totalia/labels/good%20first%20issue) and [`help wanted`](https://github.com/totaliadev/totalia/labels/help%20wanted) as they appear. Comment to claim one so two people don't work the same thing.
- **Improve the docs.** Clarity fixes, examples, and roadmap feedback all count.

Please read the [Code of Conduct](CODE_OF_CONDUCT.md) — it applies to every space in the project.

## Before you write code

- **Discuss non-trivial changes first.** For anything beyond a small fix, open an issue or Discussion before writing code, so we can agree the approach. This is especially true while the core is still being designed — a change to a shared abstraction affects every module.
- **Claim the issue.** Comment on it so it's clear you're on it.
- **Open a draft PR early.** Feedback is easier on a work in progress than on a finished branch.

## Development setup

> These steps will firm up as the solution is scaffolded; treat them as the intended workflow.

Prerequisites: the [.NET 8 SDK](https://dotnet.microsoft.com/download) (or later) and Git.

```bash
git clone https://github.com/totaliadev/totalia.git
cd totalia
dotnet build
dotnet test
```

Contributions are expected to build cleanly and keep the test suite green. Once CI is in place, every pull request runs build and tests automatically, and a green check is required before merge.

## The two conventions every abstraction must honour

These are non-negotiable design commitments, because they're what stop Totalia becoming the leaky abstraction this idea is prone to. Any new module or provider must follow both:

1. **The escape hatch.** Every abstraction must be able to return its underlying native client (for example `IAmazonS3` or `BlobServiceClient`). A consumer who needs a provider-specific feature the common surface doesn't cover must never be blocked by the abstraction.
2. **The capability contract.** Where a provider can't support an operation, the module must *declare* that rather than fail unexpectedly at runtime, so consumers can branch on capability and behaviour degrades predictably.

If a proposed abstraction can't satisfy both, that's a signal the abstraction is drawn in the wrong place — raise it in a Discussion.

## How modules and providers will be structured

Totalia is built as a shared core (the "spine") with modules plugging into it identically — same registration, options, local implementation, async shape, escape hatch and capability model for every one. That sameness is the point: it's what will make adding "provider X to module Y" closer to filling in a known template than solving an open problem.

The concrete, step-by-step "add a provider" walkthrough will be added here **once the reference storage module exists**, because it needs a real pattern to point at rather than a hypothetical one. Until then, the shape to follow is: a provider-agnostic abstraction and options in the core, a Local implementation for testing, and a per-provider implementation in its own package (`Totalia.Aws`, `Totalia.Azure`, …) — each honouring the two conventions above.

## Coding guidelines

- Target the current LTS `.NET` and enable nullable reference types.
- Async-first: `Task`-returning APIs, with a `CancellationToken` on anything that does I/O.
- Follow the standard [.NET Framework Design Guidelines](https://learn.microsoft.com/dotnet/standard/design-guidelines/) and the conventions already present in the codebase.
- Keep the public surface small and deliberate — new public API is a long-term commitment. Prefer adding to a provider implementation over widening a shared abstraction.
- Cover new behaviour with tests (xUnit). The Local provider makes most logic testable without any cloud account.
- Add XML doc comments to public types and members.

## Pull requests

- Keep PRs small and focused on a single concern — easier to review, faster to merge.
- Use a clear, descriptive title and reference the issue it closes (`Closes #123`).
- Explain the *why*, not just the *what*, in the description.
- Be patient and kind in review; assume good faith on all sides.

## Finding your way around

Work is filterable by label: `module:storage`, `module:secrets`, `module:queue` …; `provider:aws`, `provider:azure`, `provider:gcp`, `provider:local`; plus `bug`, `enhancement`, and `docs`. The [roadmap](ROADMAP.md) explains where each release is heading.

## Questions

Open a [Discussion](https://github.com/totaliadev/totalia/discussions) — no question is too small this early.
