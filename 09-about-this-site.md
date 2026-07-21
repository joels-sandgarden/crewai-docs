# About This Site

This field guide explains how CrewAI behaves at runtime. It focuses on execution semantics rather than API catalog details, so an engineer can understand the order of tasks, how agents iterate, how context flows forward, how concurrency joins, how flows schedule work, and where state lives. The guide exists to give a clear mental model of the codebase, not to restate the product docs or enumerate every option. It follows the runtime path from entry point to outcome, so the reader can trace what happens when a crew runs instead of inferring that path from the API surface. The page stays short and factual so it can frame the rest of the guide.

## Contents

| Path | Topic |
| --- | --- |
| `/01-crew-run-loop.md` | Task order and the crew execution path. |
| `/02-agent-iteration.md` | How an agent steps through work. |
| `/03-context-flow.md` | How later tasks receive earlier results. |
| `/04-concurrency-and-joins.md` | How async task work overlaps and settles. |
| `/05-flow-scheduling.md` | How flows trigger methods and listeners. |
| `/06-state-and-persistence.md` | Where flow and crew state live. |
| `/07-memory.md` | How recall and encoding fit into execution. |
| `/08-delegation-and-tools.md` | How tool calls shape a run. |
| `/09-about-this-site.md` | What this guide covers and how to read it. |

## Who this is for

This guide serves engineers who adopt CrewAI, debug crew runs, or extend the framework and want the mental model behind the API. It assumes code reading ability and focuses on behavior that explains what happens during a run: how work moves from task to task, how one agent can continue after another, how concurrent tasks join back together, and where later steps find their inputs. It also helps when a run surprises someone, because it explains how ordering, context, concurrency, and flow scheduling shape the result. It does not try to teach the public API from first principles.

## How it was made

Doc Holiday, an AI documentation writer at https://doc.holiday, wrote this guide by exploring the CrewAI source repository directly. Every page grounds its claims in actual code, with real file paths and symbol names, as of [GENERATED_FROM]. Doc Holiday uses those paths and symbols as anchors, because the guide only claims what the source shows. If the code sits in a migration, the relevant page says so and dates the observation.

## Scope notes

This guide reflects a snapshot of an actively developed codebase. The official docs at https://docs.crewai.com remain the authoritative reference for setup and configuration. When this guide differs from other descriptions, it names the runtime behavior visible in the explored snapshot and keeps the note neutral and constructive. It does not argue with the other descriptions; it records the state of the code at the moment of inspection. If later changes alter that behavior, the page should change with the code. The note stays tied to the code, not to any broader product claim.

Corrections are welcome at [CONTACT_OR_REPO_LINK].