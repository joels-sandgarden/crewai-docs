# Where State Lives

CrewAI keeps runtime state in five separate systems that often appear in the same execution path. This page maps those systems by the state they store, the moment they write, the moment they read, and the parts of the runtime that connect them.

The split is simple: memory holds learned content, flow persistence holds resumable flow state, the checkpoint system holds crew and flow snapshots, replay holds task output history, and caching holds tool results. Each system answers a different runtime question, so a clear map matters more than the individual configuration switches.

## Unified memory

CrewAI uses one `Memory` object as the runtime memory layer. That object handles save and recall in one place, and the code routes both paths through internal Flows: `crewai.memory.encoding_flow.EncodingFlow` handles save and `crewai.memory.recall_flow.RecallFlow` handles recall. The current runtime keeps memory unified rather than splitting it into separate short-term, long-term, and entity stores.

Save starts when an agent or crew hands text to memory. The `BaseAgentExecutor._save_to_memory()` path extracts relevant facts from task output, then calls `Memory.remember()` or `Memory.remember_many()`. The memory object infers scope, categories, and importance during the save pipeline, and it writes the result through a pluggable backend under `crewai.memory.storage`.

Recall starts when an agent calls recall or when the crew asks memory for context. The memory object first drains pending background writes, then it searches. Agents reach the same memory through injected tools from `crewai.tools.memory_tools.create_memory_tools()`, so recall and save stay available at the point of execution rather than as a separate setup step.

## Background writes and completion ordering

`Memory.remember_many()` does not block for the whole save pipeline. It schedules the work in a background thread and returns before the write finishes, which means a crew can finish while memory events still sit in flight.

`Crew._drain_memory_writes()` closes that gap before the code emits `CrewKickoffCompletedEvent`. That ordering keeps `MemorySaveCompletedEvent` and `MemorySaveFailedEvent` visible long enough for listeners to process them, so listener teardown does not swallow the final memory result.

## Flow persistence with `@persist`

The `@persist` decorator and the `FlowPersistence` interface form the flow-specific persistence layer in `crewai.flow.persistence.*`. The decorator only marks the flow or method for persistence; the flow engine saves state after the marked method completes.

Persisted flow state carries an `id`, and by default the flow uses SQLite when no backend appears in the flow definition. Restoring this state resumes a paused flow; it does not use the crew checkpoint path.

## Crew checkpointing in `state/`

The `state/` package owns the checkpoint story. `RuntimeState` serializes the live entity tree, the branch and parent lineage, and the execution event record, then hands that snapshot to a provider. The checkpoint config writes on the configured event, and the default trigger fires on `task_completed`.

`RuntimeState.from_checkpoint()` restores that snapshot, and `Crew.from_checkpoint()` rebuilds the live crew from it. The restore path brings back runtime state and event history together, then the crew rebinds execution context, memory views, and task state from the loaded snapshot. `Flow.from_checkpoint()` uses the same runtime snapshot path for flows, so the checkpoint story sits under one shared runtime model even when the restored object differs.

Flow kickoff keeps checkpoint restore and state-id restore separate. `Flow.kickoff()` and `kickoff_async()` reject a call that supplies both `from_checkpoint` and `restore_from_state_id`; the runtime raises `ValueError` instead of trying to merge those restore modes.

## Task execution logs and replay

Each completed task writes a record through `crewai.utilities.task_output_storage_handler.TaskOutputStorageHandler`. The stored row keeps the task identity, the raw and structured outputs, the task index, the original inputs, and whether the run came from replay. `Crew` uses that store as its audit trail for task execution.

`Crew.replay(task_id, ...)` reads those stored outputs back, finds the requested task, restores every earlier task output onto the crew, and starts execution again from that point. The earlier outputs stay in use, the later tasks run again, and the storage row for the replayed run marks `was_replayed=True`. In other words, replay does not rewind the whole crew; it reruns the middle with the earlier part loaded from storage.

## Tool-result cache

Tool-result caching stays opt in. `crewai.agents.cache.cache_handler.CacheHandler` stores results by tool name and input, `crewai.tools.cache_tools.cache_tools.CacheTools` exposes a cache-read tool, and `crewai.llms.cache` only marks cache breakpoints for provider adapters. The code does not promise a universal LLM cache, so this layer works as a helper around tool execution rather than as a separate persistence system.

## Comparison map

| System | What it stores | Written when | Read when | Composes with |
| --- | --- | --- | --- | --- |
| `Memory` | Learned facts, decisions, and other recalled content | During `remember()` / `remember_many()` and after agent execution via `BaseAgentExecutor._save_to_memory()` | During `recall()` and through injected memory tools | Agent tools, `EncodingFlow`, `RecallFlow`, crew memory views |
| Flow persistence | Flow state snapshots with an `id` | After a persisted flow method completes | When a paused flow resumes from persisted state | `FlowPersistence`, `@persist`, `SQLiteFlowPersistence`, `Flow.from_pending()`, `Flow.resume()` |
| Checkpointing | `RuntimeState`, event history, lineage, and checkpoint fields | When a configured checkpoint event fires | When `RuntimeState.from_checkpoint()`, `Crew.from_checkpoint()`, or `Flow.from_checkpoint()` loads a snapshot | `state/provider/*`, `CheckpointConfig` |
| Task replay | Task outputs, inputs, and replay status | After each task completes through `TaskOutputStorageHandler` | When `Crew.replay(task_id, ...)` reloads earlier outputs | `TaskOutputStorageHandler`, `KickoffTaskOutputsSQLiteStorage` |
| Tool-result caching | Cached tool outputs keyed by tool input | When a tool call writes to `CacheHandler` | When `CacheTools.hit_cache()` reads a cached value | Tool execution path, `crewai.llms.cache` markers |

## Where to look in the code

- `lib/crewai/src/crewai/memory/unified_memory.py` — `Memory.remember()`, `Memory.remember_many()`, `Memory.recall()`, `Memory.drain_writes()`
- `lib/crewai/src/crewai/memory/encoding_flow.py` and `lib/crewai/src/crewai/memory/recall_flow.py` — `EncodingFlow`, `RecallFlow`
- `lib/crewai/src/crewai/crew.py` — `Crew._drain_memory_writes()`, `Crew.replay()`, `Crew.from_checkpoint()`
- `lib/crewai/src/crewai/state/runtime.py` — `RuntimeState`, `RuntimeState.checkpoint()`, `RuntimeState.from_checkpoint()`
- `lib/crewai/src/crewai/flow/persistence/decorators.py` and `lib/crewai/src/crewai/flow/persistence/sqlite.py` — `persist`, `PersistenceDecorator.persist_state`, `SQLiteFlowPersistence`
- `lib/crewai/src/crewai/utilities/task_output_storage_handler.py` and `lib/crewai/src/crewai/memory/storage/kickoff_task_outputs_storage.py` — `TaskOutputStorageHandler`, `KickoffTaskOutputsSQLiteStorage`