# agent-harness

**Defining the behaviors that LLM frameworks leave undefined.**

We're building trillion-parameter models and jamming context into them with string concatenation.

Every agent framework gives you the same thing: a loop. Call the model, parse tool calls, execute tools, feed results back, repeat. They all nail this part.

But here's what they leave undefined:

- When does the agent stop based on what's actually happened?
- What context gets injected where?
- How do tool outputs render for models vs UIs?
- How do you enforce tool behaviors?
- How do you remind the model of constraints?

The agent harness defines these behaviors.

---

## Injection Points

Every conversation has the same shape:

```
┌─────────────────────────────────────────────────────────┐
│ SYSTEM MESSAGE                                          │ ← injection point
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ USER MESSAGE                                            │ ← injection point
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ ASSISTANT                                               │
│   ┌───────────────────────────────────────────────────┐ │
│   │ Tool Call                                         │ │
│   └───────────────────────────────────────────────────┘ │
│   ┌───────────────────────────────────────────────────┐ │
│   │ Tool Response                                     │ │ ← injection point
│   └───────────────────────────────────────────────────┘ │
│   ... more calls ...                                    │
│   Final response                                        │
└─────────────────────────────────────────────────────────┘
```

Frameworks define how messages flow. The harness defines what gets injected at each point, when, and why.

---

## The Seven Behaviors

### 1. Tool Output Protocol

Tools serve two consumers: UIs and models. UIs want structured JSON. Models want whatever format aids comprehension.

```
┌─────────────────────────────────────────┐
│ Structured Data (JSON)                  │  → for UIs, logging, debugging
├─────────────────────────────────────────┤
│ Model Rendering                         │  → format optimized for LLM
├─────────────────────────────────────────┤
│ Attached Reminders                      │  → context to inject with result
└─────────────────────────────────────────┘
```

One tool output, multiple renderings.

### 2. Conversation State

Treat conversation history as queryable state. Not just a list of messages - an event stream with views.

- How many times has this tool failed?
- What has the model already tried?
- How much context has accumulated?
- Is the model stuck in a loop?

Views over the stream, not scattered bookkeeping.

### 3. System Reminders

Context that gets injected at injection points. Three levels:

**System-level**: Seed the system message with awareness that reminders exist. Include a few-shot example so the model knows the format and pays attention.

**Message-level**: Reminders that attach to user messages or tool responses.

**Tool-level**: Reminders bound to specific tools. Only surfaces when that tool is called.

### 4. Stop Conditions

When does the agent stop? Define it explicitly:

- **Turn limit**: Stop after N turns
- **Token budget**: Stop when context exceeds threshold
- **Task completion**: Stop when a condition is met
- **Error threshold**: Stop after N consecutive failures
- **Custom rules**: Any condition over conversation state

Integrated with conversation state, not isolated flags.

### 5. Tool Enforcement Rules

Rules that govern tool behavior:

- **Sequencing**: "Always read a file before editing it"
- **Confirmation**: "Confirm with user before deleting files"
- **Rate limiting**: "Max 3 retries per tool per turn"
- **Auto-actions**: "When context exceeds 80%, trigger compaction"

These aren't suggestions to the model. They're enforced by the harness.

### 6. Injection Queue

Reminders accumulate. A queue manages them:

- Prioritization (safety reminders first)
- Batching (group related context)
- Deduplication (don't repeat yourself)

When an injection point arrives, the queue flushes strategically.

### 7. Hooks

Plugin system for everything. Custom stop conditions? Hook. Custom rendering? Hook. Custom injection logic? Hook.

The harness provides structure. Hooks provide flexibility.

---

## Architecture

A harness guides without replacing. It wraps the agent loop, observes the conversation, enforces rules, injects context.

```
┌─────────────────────────────────────────────────────────┐
│                    Agent Framework                      │
└─────────────────────┬───────────────────────────────────┘
                      │ conversation
                      ▼
┌─────────────────────────────────────────────────────────┐
│                    Agent Harness                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐  │
│  │  State   │→ │  Rules   │→ │  Queue   │→ │Renderer │  │
│  └──────────┘  └──────────┘  └──────────┘  └─────────┘  │
└─────────────────────┬───────────────────────────────────┘
                      │ enriched context
                      ▼
┌─────────────────────────────────────────────────────────┐
│                      LLM API                            │
└─────────────────────────────────────────────────────────┘
```

The goal: framework-agnostic. Should work with LangChain, CrewAI, Vercel AI SDK, or raw API calls.

---

## Status

**Early development.** This is a specification and work in progress.

We're looking for collaborators to help shape and build this.

---

## Get Involved

- **[Discussions](https://github.com/Michaelliv/agent-harness/discussions)** - Share ideas, ask questions, debate architecture
- **[Issues](https://github.com/Michaelliv/agent-harness/issues)** - Propose features or report problems with the design
- **Star the repo** - Help others find this project

### Open Questions

1. What's the right event schema for conversation state?
2. How should rules be expressed? DSL vs code?
3. What's the integration surface for existing frameworks?
4. TypeScript, Python, or both?

If you have opinions, [start a discussion](https://github.com/Michaelliv/agent-harness/discussions/new).

---

## Implementations

- [agent-harness-ai-sdk](https://github.com/Michaelliv/agent-harness-ai-sdk) - Vercel AI SDK implementation (in progress)

---

## Related Reading

- [The Agent Harness](https://michaellivs.com/blog/agent-harness) - The blog post explaining this project

---

## License

MIT
