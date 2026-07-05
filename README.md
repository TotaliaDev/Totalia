# Totalia

**Write once. Run on any cloud.** A .NET library for building applications against one consistent, in-process surface — storage, secrets, queues and more — where the cloud provider behind each abstraction is a configuration choice, not a code change. No sidecar, no runtime.

![Status](https://img.shields.io/badge/status-design%20phase%20%C2%B7%20pre--release-orange)

<!-- Add these once CI runs and the first package is published:
[![Build](https://img.shields.io/github/actions/workflow/status/totaliadev/totalia/ci.yml?branch=main)](https://github.com/totaliadev/totalia/actions)
[![NuGet](https://img.shields.io/nuget/v/Totalia.svg)](https://www.nuget.org/packages/Totalia)
-->

> **This repo is at the design stage — there is no code to use yet.** It currently holds the vision, the [roadmap](ROADMAP.md), and the design decisions behind Totalia. The first release (the core plus storage and secrets, on AWS, Azure and Local) is being built in the open. If the idea resonates, the best time to shape it is now — see [Getting involved](#getting-involved).

---

## The idea

Most cloud applications are written directly against a single provider's SDK — `IAmazonS3`, `BlobServiceClient`, `SecretsManagerClient`. That's fine until you need to move a workload, run the same code against two clouds, or test locally without standing up real infrastructure. At that point the provider-specific surface is woven through your business logic, and unpicking it is expensive.

Totalia's aim is to give you a small set of provider-agnostic abstractions — `ICloudStorage`, `ICloudSecrets`, `ICloudQueue` — that your business logic depends on instead. Which provider runs behind them becomes a configuration choice. Swap the provider and your application code doesn't move.

## What makes Totalia different

Several libraries already abstract *one* service — storage, say. The point of Totalia isn't any single abstraction; it's that **all of them share one model**: one registration call, one options pattern, one local-development implementation, one async convention, one escape hatch to the native SDK, and one capability contract. That consistency across every service is the whole reason the project exists — and it's what makes it feel like one coherent thing rather than a bag of unrelated wrappers.

## The experience we're building toward

> ⚠️ **Illustrative only — not yet implemented.** This is the developer experience Totalia is being designed around, shown so you can picture the goal. No package is published yet.

```csharp
// Register once. The provider is a config choice.
builder.Services.AddTotalia(options => options.UseAws());

// Business logic depends only on the abstraction — it has no idea which cloud it's on.
public class ReportService(ICloudStorage storage, ICloudSecrets secrets)
{
    public async Task PublishAsync(string name, Stream content, CancellationToken ct)
    {
        var apiKey = await secrets.GetAsync("reporting-api-key", ct);
        await storage.WriteAsync($"reports/{name}", content, ct);
    }
}

// Moving from AWS to Azure is one line — UseAws() becomes UseAzure(). Nothing above changes.
```

## Planned scope

The first milestone is deliberately narrow: the core plus **two** differently-shaped modules, which is the minimum needed to prove the shared model generalises.

| Module | Abstraction | Target |
|---|---|---|
| Storage | `ICloudStorage` | v0.1 (in progress) |
| Secrets | `ICloudSecrets` | v0.1 (in progress) |
| Queue | `ICloudQueue` | planned |
| Email | `ICloudEmail` | planned |
| Scheduler | `ICloudScheduler` | planned |

Providers for v0.1: **AWS**, **Azure**, and **Local** (in-memory / on-disk, for tests and offline development). **Google Cloud** is planned as a third provider once the model has proven itself across two. The full plan lives in the [roadmap](ROADMAP.md).

## Design principles

These are the constraints that keep Totalia from becoming the leaky, lowest-common-denominator abstraction this idea is prone to:

- **Expose the common 80%, not the union of everything.** The portable surface stays small and predictable.
- **Always provide an escape hatch.** Every abstraction can return its underlying native client, so you're never trapped when you need a provider-specific feature.
- **Declare capabilities, don't hide them.** Where providers differ, each module reports what it supports, so behaviour degrades predictably rather than throwing surprises.
- **No sidecar, no runtime.** Totalia is a plain library. If you can `dotnet add package`, you can use it.
- **DI-native, async-first, local-first.** One entry point, the options pattern throughout, `Task`-based APIs with `CancellationToken`, and an in-memory implementation for every abstraction.

## How Totalia relates to other projects

It's worth being honest about the neighbours:

- **[Dapr](https://dapr.io)** offers building-block abstractions across providers, but runs as a **sidecar** — a separate process with its own operational footprint. Totalia is a plain in-process library with nothing to operate.
- **[FluentStorage](https://github.com/robinrodricks/FluentStorage)** is an excellent, battle-tested library for blob storage and messaging. Totalia's aim is broader: one consistent model across more service types than storage and queues alone.
- **[.NET Aspire](https://github.com/dotnet/aspire)** solves local orchestration and developer experience and deliberately does *not* abstract providers. Totalia is complementary — you can run a Totalia-based app under Aspire and get both.

If one of the above fully meets your needs, use it. Totalia is for teams who want a single, cohesive, sidecar-free surface across several cloud services in .NET.

## Project status

Design and roadmap are in place. The v0.1 core (the shared "spine") and the storage and secrets modules are being built. Nothing is published to NuGet yet, and the public API may change freely until 1.0. Follow the [roadmap](ROADMAP.md) for what's next.

## Getting involved

At this stage the most valuable contributions are shaping the design and helping build the first release. If the idea interests you: open a discussion, weigh in on the [roadmap](ROADMAP.md), or watch for issues labelled [`good first issue`](https://github.com/totaliadev/totalia/labels/good%20first%20issue) and [`help wanted`](https://github.com/totaliadev/totalia/labels/help%20wanted) as v0.1 work gets broken down. See [CONTRIBUTING](CONTRIBUTING.md) for how the project is structured and the conventions every contribution follows.

## The name

Totalia (*toh-TAH-lee-uh*) — for **total** coverage across your cloud stack, and for making the common surface the whole story.

## License

[MIT](LICENSE) — free for commercial and open-source use alike.
