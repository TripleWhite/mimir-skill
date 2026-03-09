---
name: mimir-memory
description: >
  Long-term memory for AI agents powered by Mimir (knowledge graph + vector + BM25).
  Use when: (1) The user asks about past conversations, people, events, or preferences,
  (2) The user says "remember this" or shares important facts to save,
  (3) The user references something discussed before ("remember when...", "what did I say about..."),
  (4) The user wants to forget or delete stored memories,
  (5) Auto-recalled memories are insufficient and a deeper search is needed,
  (6) The user asks a time-specific question about the past ("last week", "in January").
  Available tools: mimir_search (active memory lookup), mimir_store (save facts), mimir_forget (delete memories).
  Auto-recall injects basic context automatically before each conversation — use mimir_search for deeper queries.
---

# Mimir Memory

## How Memory Works

- **Auto-recall**: Before each conversation, relevant memories are silently injected as `<memories>` context. No action needed.
- **mimir_search**: Actively search memory when auto-recall is insufficient.
- **mimir_store**: Save important facts, preferences, or notes the user wants remembered.
- **mimir_forget**: Look up and preview memories for deletion.

## Using mimir_search

### Query Crafting

Be specific. Include names, dates, and topic keywords.

```
GOOD: "Arthur's Chrome extension project in March"
BAD:  "that project"

GOOD: "Caroline's birthday gift ideas"
BAD:  "gift ideas"
```

### memory_types Selection

| Goal | memory_types | Example query |
|------|-------------|---------------|
| Specific facts/events | `["event_log"]` | "What did I eat last Tuesday?" |
| People/places/things | `["entity"]` | "Who is Caroline?" |
| How things connect | `["relation"]` | "How do Arthur and Caroline know each other?" |
| Full conversation context | `["episode"]` | "Summarize our Chrome extension discussion" |
| Best general results | `["event_log", "entity", "relation"]` | "Tell me about the Japan trip" |
| Broad exploratory | *(omit)* | When unsure what type to look for |

### Time Filtering

When the user mentions time, convert to ISO 8601 and pass `start_time`/`end_time`:

- "last week" → 7 days ago to now
- "in March 2025" → `2025-03-01` to `2025-03-31`
- "yesterday" → yesterday 00:00 to 23:59

### Parameters

```
mimir_search({
  query: string,          // Required. Specific search query.
  memory_types?: string[],// Optional. Filter: "event_log", "entity", "relation", "episode", "raw_doc", "foresight"
  start_time?: string,    // Optional. ISO 8601 start.
  end_time?: string,      // Optional. ISO 8601 end.
  top_k?: number          // Optional. Max results (default 10, max 30).
})
```

## Using mimir_store

Save when the user explicitly asks to remember something, or shares clearly important personal information (preferences, plans, decisions).

```
mimir_store({ content: "Arthur prefers dark roast coffee, no sugar." })
```

Keep content atomic — one fact per call. Include the person's name for attribution.

## Installation

```bash
# Zero-config setup (anonymous device, no signup needed)
npx memory-mimir init

# Or with existing API key
npx memory-mimir setup --api-key sk-mimir-xxx
```

Restart the AI agent after setup. Memory is active immediately.
