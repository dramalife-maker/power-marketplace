---
name: dev:show-me
description: Help the user understand the current topic visually with concise diagrams, code-shape sketches, and polished visual artifacts. Use when the user asks to visualize, diagram, sketch, or "show me" a concept, architecture, flow, comparison, or piece of code.
---

Help the user understand the current topic of conversation visually. Skip the preamble and keep prose brief. Pick the smallest view that makes the key point clear.

## Lightweight text views (default first choice)

Use these when a plain fenced block already makes the point — no tool call needed.

- **Pseudocode** for logic or an algorithm:

```text
on(save)
  if content is unchanged
    return cached result
  write new content
  return fresh result
```

- **Call tree** for runtime control flow:

```text
submitForm
  createSession
    persistPrompt
    launchAgent
  navigateToSession
```

- **Component tree** for UI structure, including state and module boundaries that matter:

```tsx
<SessionPage> (apps/example/src/routes/session.tsx)
  useSessionEvents()
  <SessionToolbar>
    <RunSkillButton> (packages/ui)
```

- **Shallow file tree** for file responsibility or a broad refactor:

```text
src/
├── commands/       # parses user actions
├── sessions/       # owns session state
└── transport/      # sends API requests
```

- **`diff`** when the point is what changes and the surrounding shape already exists. Match the diff shape to the topic (component tree, file layout, call tree, or control flow).

```diff
 <SessionPage>
   useSessionEvents()
   <SessionToolbar>
+    <RunSkillButton />
```

- Show the whole block instead of a diff when most of it is new, when omitted context would hide ownership or order, or when the user needs a copyable target shape.

## Diagrams — `diagram-design` is the default

Anything past the plain-text views above — component interaction, control flow, data flow, architecture, a chart, a mockup — **defaults to the `diagram-design:diagram-design` skill.** This applies regardless of whether the visual needs to look polished or is meant to be quick and disposable; do not reach for a hand-rolled Mermaid block or raw HTML just because the need feels lightweight. `diagram-design` draws a materially cleaner, more consistent result and should be the first move every time a real diagram is needed.

1. **Check availability first.** Look at the skills currently available to you (the system's skill listing). If `diagram-design:diagram-design` is present, use it as the default path below. If it is not installed, use the fallback path and suggest the user install it.
2. **Default path — `diagram-design` is installed:**
   - Invoke `diagram-design:diagram-design` and describe what to draw, picking the nearest of its 38 visual types (architecture, flowchart, sequence, state machine, ER, timeline, swimlane, quadrant, radar, loop, nested, tree, org chart, layer stack, Venn, pyramid/funnel, treemap, bar/line/Gantt/scatter, high-level, process, medallion, data flow, DP integration/security matrix, Sankey, fishbone, Wardley, kanban, user journey, deployment, dependency graph, UML class, story map, DB schema).
   - Let it run its own style-guide gate normally on first use in a project — don't try to skip or pre-answer that prompt on the user's behalf.
   - If the visual genuinely doesn't fit any of the 38 types (a bespoke UI mockup, dashboard, or interactive comparison), author a focused HTML file yourself: match the product's real colors, type, and spacing, use real labels and data, and support desktop and mobile. If an `artifact-design` skill is available in this environment, load it first for design guidance. Publish the result with the `Artifact` tool when that tool is available (Claude Code); otherwise just write the HTML file to disk and tell the user its path — don't call a publishing tool that isn't in your current tool list.
   - If a Mermaid diagram from earlier in the conversation turned out worth keeping, offer to redraw it with `diagram-design:import-mermaid` rather than leaving it as plain Mermaid.
3. **Fallback path — `diagram-design` is not installed:**
   - Use a Mermaid block for component interaction, control flow, or data flow. For anything denser than Mermaid can express, hand-roll an HTML file the same way as the non-matching-type case above — design guidance from `artifact-design` if it's available, published via `Artifact` if that tool exists in this environment, otherwise written to disk with the path reported to the user.
   - Tell the user `diagram-design` would produce a cleaner result for this and suggest installing it (surface it via `SuggestSkills`, or point them to the plugin marketplace entry) — once, not on every single visual in the conversation.

Neither `artifact-design` nor the `Artifact` tool is guaranteed to exist outside Claude Code (e.g. Cursor running this same plugin). Always check what's actually in your current skill/tool list before reaching for either — never assume they're present just because they're mentioned here.

## Guidance

Place each visual next to the short text it supports. Keep only the calls, files, props, states, and boundaries needed to answer the user's current question or the options to resolve the current discussion point.

You may use one of these, you may use several, it is unlikely you will use all of them. Use your judgement and don't overwhelm the user.
