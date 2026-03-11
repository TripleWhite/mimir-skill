---
name: mimir-memory
description: >
  Long-term memory for AI agents powered by Mimir (knowledge graph + vector + BM25).
  Use when: (1) The user asks about past conversations, people, events, or preferences,
  (2) The user says "remember this" or shares important facts to save,
  (3) The user references something discussed before ("remember when...", "what did I say about..."),
  (4) The user wants to forget or find stored memories about a topic,
  (5) Auto-recalled memories are insufficient and a deeper search is needed,
  (6) The user asks a time-specific question about the past ("last week", "in January").
  Available tools: mimir_search (active memory lookup), mimir_store (save facts), mimir_forget (preview matching memories).
  Auto-recall injects basic context automatically — use mimir_search for deeper or broader queries.
---

# Mimir Memory

## First Conversation (Onboarding)

If no `<memories>` block is present in this conversation, the user just installed Mimir.
Welcome them warmly and guide them to try it out. Example response:

---
记忆已就绪！我现在可以跨对话记住你告诉我的事情了。

试试看：
- **介绍自己** — 告诉我你的名字、职业、兴趣，我会记住
- **让我记住什么** — 比如"记住我喜欢深色模式"
- **下次对话验证** — 重启后问"你还记得我吗？"

你聊天的重要内容我也会自动捕捉，不用每次都说"记住"。
---

Adapt the language to match the user's language. Keep it short and actionable.
Do NOT repeat this in subsequent conversations where `<memories>` is present.

## How Memory Works

The plugin has two automatic behaviors and three manual tools:

### Automatic (no action needed)

- **Auto-recall**: Before each conversation, relevant memories are silently injected as a `<memories>` block. It searches **event_log, entity, and relation** types only, returning up to 15 results. It does NOT search episodes or raw documents.
- **Auto-capture**: After each conversation ends, new messages are automatically ingested into memory. You do NOT need to manually save everything the user says — important content is captured automatically.

### Manual Tools

- **mimir_search**: Actively search memory. Use when auto-recall is insufficient — especially for episodes, full conversation context, or specific time ranges.
- **mimir_store**: Save an explicit atomic fact or preference. Use sparingly — auto-capture already handles most content.
- **mimir_forget**: Preview memories matching a query. NOTE: Actual deletion is not yet supported — this only shows what would be affected.

## When to Use mimir_search (vs relying on auto-recall)

Auto-recall covers basic facts, entities, and relations. Use mimir_search when:

| Scenario | Why auto-recall isn't enough |
|----------|------------------------------|
| "Summarize our Chrome extension discussion" | Auto-recall doesn't search **episodes** |
| "What did I say last Tuesday?" | Auto-recall may not apply **time filters** |
| "Tell me everything about the Japan trip" | Need broader search with multiple memory types |
| Auto-recalled `<memories>` don't answer the question | Need different query terms or types |

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

## When to Use mimir_store (vs relying on auto-capture)

Auto-capture saves the full conversation after each session. Use mimir_store ONLY for:

- **Explicit requests**: User says "remember that I prefer dark mode"
- **Atomic facts worth highlighting**: Important preferences, decisions, or plans that might get buried in a long conversation

Do NOT use mimir_store to save things the user already said in conversation — auto-capture handles that.

```
mimir_store({ content: "Arthur prefers dark roast coffee, no sugar." })
```

Keep content atomic — one fact per call. Include the person's name for attribution.

## Using mimir_forget

Preview memories matching a topic the user wants to forget. Deletion is NOT yet implemented — inform the user that you can show what's stored but cannot delete it yet.

```
mimir_forget({ query: "my old phone number" })
```

## Installation

```bash
# With invite code (closed beta, single-use)
npx memory-mimir init --code XXXXXX

# Or with existing API key
npx memory-mimir setup --api-key sk-mimir-xxx
```

Restart the AI agent after setup.
