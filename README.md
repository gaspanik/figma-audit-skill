# figma-audit — Figma AI-Readiness Audit Skill

A Claude Code skill that evaluates whether a Figma frame is structured in a way that allows AI to implement it accurately. Pass a Figma URL and get a scored report across 4 dimensions, with concrete suggestions to improve the design before handing it off to an AI coding agent.

---

## What this is

AI coding agents read Figma files the same way they read code — structure matters as much as appearance. A frame with unnamed layers, hardcoded colors, and no auto-layout will produce worse code than one built with design tokens and components, even if they look identical on screen.

This skill makes that gap visible. It runs two Figma API calls in parallel (`get_design_context`, `get_metadata`), analyzes the result across 4 dimensions, and returns a 20-point report with specific, actionable fixes.

```
Figma URL  →  figma-audit  →  AI-readiness score (X / 20)
                           →  Dimension breakdown
                           →  Improvement suggestions per layer
```

---

## The 4 dimensions

| # | Dimension | What is checked |
|---|-----------|-----------------|
| A | **Layer naming** | Meaningful role-based names vs. `Frame 123`, `Group 456`, duplicate siblings |
| B | **Auto-layout** | Proper padding/gap usage vs. absolute positioning and spacer frames |
| C | **Variables & tokens** | `var(--token)` in generated code vs. raw hex colors and fixed px values |
| D | **Component structure** | Instances and variants vs. manually duplicated frames |

Each dimension is scored 0–5. Total: **20 points**.

---

## Repo structure

```
skills/
  figma-audit/
    SKILL.md          — skill definition loaded by Claude Code
```

---

## Getting started

**1. Clone this repo**

```bash
git clone https://github.com/gaspanik/figma-audit-skill
```

**2. Install the skill into Claude Code**

Copy the skill directory into your Claude Code skills folder:

```bash
cp -r skills/figma-audit ~/.claude/skills/
```

**3. Verify the Figma MCP is connected**

This skill requires the official [Figma MCP server](https://github.com/figma/mcp-server-guide). Confirm it is connected in your Claude Code settings before running the skill.

**4. Run the audit**

Invoke the skill with a slash command or just describe what you want in natural language — both work:

```
/figma-audit https://www.figma.com/design/<fileKey>/...?node-id=1-2
```

```
このFigmaフレームをチェックして: https://www.figma.com/design/...
```

```
Can you audit this Figma frame? https://www.figma.com/design/...
```

The report is written in whatever language you are using in the conversation — no configuration needed.

You will receive a report in this format:

```
## Figma AI-Readiness Audit Report

Overall score: 14 / 20

| Dimension          | Score | Rating |
|--------------------|-------|--------|
| A. Layer naming    | 4 / 5 | 🟢 Good |
| B. Auto-layout     | 3 / 5 | 🟡 Room for improvement |
| C. Variables/tokens| 2 / 5 | 🔴 Needs fixing |
| D. Component struct| 5 / 5 | 🟢 Good |

### Improvement suggestions
...
```

---

## Agent settings

- Use the most capable Claude model available. The skill calls two Figma API tools in parallel and synthesizes a multi-section report — a stronger model produces more precise layer-level suggestions.
- The `SKILL.md` format and `allowed-tools` frontmatter are Claude Code-specific. To use this with another agent (Cursor, Windsurf, etc.), copy the prompt body from `SKILL.md` into that agent's rule format — the audit logic itself is plain markdown and transfers without modification.

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `fileKey or nodeId could not be extracted` | Make sure the URL points to a specific frame, not the file root. The URL must contain `?node-id=X-Y`. |
| `get_design_context returned an error` | Check that your Figma MCP is connected and has access to the file. Private files require the file owner to grant access. |
| `Passed a component URL instead of a frame` | Right-click the frame in Figma → Copy link → use that URL. Component URLs and file-root URLs are not supported. |
| Report is in the wrong language | The skill mirrors the conversation language. Switch your message language and re-run. |

---

Built by Masaaki Komori - [@cipher](https://x.com/cipher) · Skill for [Claude Code](https://claude.ai/code) + [Figma MCP](https://github.com/figma/mcp-server-guide)
