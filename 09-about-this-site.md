# About This Site

This field guide explains how CrewAI behaves at runtime. It focuses on execution semantics rather than API catalog details, so an engineer can understand the order of tasks, how agents iterate, how context flows forward, how concurrency joins, how flows schedule work, and where state lives. The goal is a mental model that makes the codebase easier to adopt, debug, and extend without reading it end to end.

## Contents

- [`/01-crew-run-loop.md`](./01-crew-run-loop.md) — Task order and the crew execution path.
- [`/02-agent-iteration.md`](./02-agent-iteration.md) — How an agent steps through work.
- [`/03-context-flow.md`](./03-context-flow.md) — How later tasks receive earlier results.
- [`/04-concurrency-and-joins.md`](./04-concurrency-and-joins.md) — How async task work overlaps and settles.
- [`/05-flow-scheduling.md`](./05-flow-scheduling.md) — How flows trigger methods and listeners.
- [`/06-state-and-persistence.md`](./06-state-and-persistence.md) — Where flow and crew state live.
- [`/07-memory.md`](./07-memory.md) — How recall and encoding fit into execution.
- [`/08-delegation-and-tools.md`](./08-delegation-and-tools.md) — How tool calls shape a run.
- [`/09-about-this-site.md`](./09-about-this-site.md) — What this guide covers and how to read it.

## Who this is for

This guide serves engineers who adopt CrewAI, debug crew runs, or extend the framework and want the mental model behind the API. It assumes code reading ability and keeps the focus on behavior rather than configuration catalogs.

## How it was made

Doc Holiday, an AI documentation writer at https://doc.holiday, wrote this guide by exploring the CrewAI source repository directly. Every page grounds its claims in actual code, with real file paths and symbol names, as of [GENERATED_FROM]. When the code sits in a migration, the relevant page says so and dates the observation.

## Scope notes

This guide reflects a snapshot of an actively developed codebase. The official docs at https://docs.crewai.com remain the authoritative reference for setup and configuration. When this guide differs from other descriptions, it states the runtime behavior observed in the explored snapshot in a neutral, constructive way.

Corrections are welcome at [CONTACT_OR_REPO_LINK].