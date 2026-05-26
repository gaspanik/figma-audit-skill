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

### Figma MCP tool not found / `get_design_context` or `get_metadata` fails to resolve

The Figma MCP server can be connected in two ways, and the internal tool names Claude Code uses differ between them. This skill calls `get_design_context` and `get_metadata` in parallel — if either cannot be resolved, the audit will not run.

#### Installation method A — Plugin (Claude.ai / Claude Code desktop)

Install the Figma MCP via the Claude Code plugin marketplace or by adding it from the Claude.ai integrations panel. When installed this way, Claude Code registers the tools under a plugin-namespaced prefix:

```
mcp__plugin_figma_figma__get_design_context
mcp__plugin_figma_figma__get_metadata
...
```

The skill's `SKILL.md` refers to these tools without a namespace, and Claude Code resolves them automatically to the plugin-prefixed versions.

#### Installation method B — Direct settings configuration (settings.json)

Add the Figma MCP server manually to your Claude Code settings file:

```json
// .claude/settings.json  (project-level)
// or ~/.claude/settings.json  (user-level)
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": ["-y", "@figma/mcp-server"]
    }
  }
}
```

With this approach the tools are registered under a shorter prefix:

```
mcp__figma__get_design_context
mcp__figma__get_metadata
...
```

#### If the skill still fails to call the Figma tools

1. Open Claude Code and type `/mcp` or check **Settings → MCP** to confirm the Figma server appears in the connected-servers list.
2. Ask Claude directly: *"What Figma MCP tools are available?"* — the response will show the exact prefixed names that are active.
3. If both a plugin and a settings entry exist at the same time, two sets of tools will be registered and may conflict. Remove one of them.
4. Restart Claude Code after any settings change.

---

### Figma MCPツールが見つからない / `get_design_context` や `get_metadata` が解決されない場合（日本語）

Figma MCPサーバーの接続方法は2通りあり、Claude Codeが内部で使うツール名がそれぞれ異なります。このスキルは `get_design_context` と `get_metadata` を並列で呼び出します。どちらか一方でも解決できない場合、監査は実行されません。

#### 方法A — プラグイン（Claude.ai / Claude Codeデスクトップ）

Claude Codeのプラグインマーケットプレイスや、Claude.aiのインテグレーションパネルからFigma MCPをインストールした場合、ツール名はプラグイン用のプレフィックスで登録されます。

```
mcp__plugin_figma_figma__get_design_context
mcp__plugin_figma_figma__get_metadata
```

スキルの `SKILL.md` ではプレフィックスなしで書かれていますが、Claude Codeが自動解決します。

#### 方法B — settings.json への直接記述

Claude Codeの設定ファイルにFigma MCPサーバーを手動で追加した場合：

```json
// .claude/settings.json（プロジェクト）または ~/.claude/settings.json（ユーザー）
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": ["-y", "@figma/mcp-server"]
    }
  }
}
```

この場合、ツール名は短いプレフィックスで登録されます。

```
mcp__figma__get_design_context
mcp__figma__get_metadata
```

#### それでもFigmaツールの呼び出しに失敗する場合

1. Claude Codeで `/mcp` と入力するか、**Settings → MCP** を開き、Figmaサーバーが「接続済み」の一覧に表示されていることを確認する。
2. Claudeに「使えるFigma MCPツールは何ですか？」と聞くと、現在アクティブなプレフィックス付きのツール名一覧が返ってきます。
3. プラグインと settings.json の両方に同時に設定が存在すると、2セットのツールが登録されて競合することがあります。どちらか一方を削除してください。
4. 設定を変更したら、必ずClaude Codeを再起動してください。

---

## Related

- [figma-component-audit-skill](https://github.com/gaspanik/figma-component-audit-skill) — the companion skill for auditing **components** (variants, slots, component properties) rather than frames

---

Built by Masaaki Komori - [@cipher](https://x.com/cipher) · Skill for [Claude Code](https://claude.ai/code) + [Figma MCP](https://github.com/figma/mcp-server-guide)
