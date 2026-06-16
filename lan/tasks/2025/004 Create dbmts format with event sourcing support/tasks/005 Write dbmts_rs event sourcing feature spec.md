---
parent: '[[004 Create dbmts format with event sourcing support]]'
spawned_by: '[[004 Write dbmts_rs spec]]'
context_type: task
status: done
---

Parent: [004 Create dbmts format with event sourcing support](../004%20Create%20dbmts%20format%20with%20event%20sourcing%20support.md)

Spawned by: [004 Write dbmts_rs spec](004%20Write%20dbmts_rs%20spec.md)

Spawned in: [<a name="spawn-task-735a71" />^spawn-task-735a71](004%20Write%20dbmts_rs%20spec.md#spawn-task-735a71)

# 1 Journal

2025-10-10 Wk 41 Fri - 08:52 +03:00

|Table|Purpose|
|-----|-------|
|`{table_name}_hist`|Contains historical objects, with a unique `id` for an `obj_id` at a point in history (`span` and `frame`). This is the result of the accumulation of events as specified by the configured columns in the `{table_name}_diffs` table.|
|`{table_name}_events`|Software uses this to write new events They are composed of event data + the diff portion the user wrote.|
|`{table_name}_events_grouped`|Events as grouped by `span` and `frame` and freezing prior span events for a new frame created.|
|`{table_name}_events_grouped_partial`|For queries that filter `{table_name}_events_grouped` to view a partial history of only the partial events.|
|`{table_name}_hist_partial`|Like `{table_name}_hist`, but only uses the events in `{table_name}_events_grouped_partial`|

2025-10-10 Wk 41 Fri - 09:26 +03:00

From [001 Create coin table events to experiment with aggregation being in views](https://github.com/deltatraced/delta-trace/blob/webview/lan/protos/2025/001%20Rust%20Diesel%20Event%20Sourcing/tasks/2025/000%20Implement%20the%20Event%20Accumulator/tasks/001%20Create%20coin%20table%20events%20to%20experiment%20with%20aggregation%20being%20in%20views.md) in delta-trace,

|Event Action|Description|
|------------|-----------|
|insert `s` `f` `diff_id`|Insert a new object specified by `diff_id` at frame `f` span `s`.|
|update `s` `f` `diff_id`|Update an object specified by `diff_id`  at frame `f` span `s`.|
|delete `s` `f` `diff_id`|Delete an object specified by `diff_id` at frame `f` span `s`.|
|open `s` `f`|Creates a new frame `f` with span `s`.|
|close `s` `f`|Closes a frame `f` with span `s`. Events should no longer append there.|
|reopen `s` `f`|Reopens a frame `f` with span `s`. Events could append there again.|
