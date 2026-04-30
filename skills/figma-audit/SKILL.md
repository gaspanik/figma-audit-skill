---
name: figma-audit
description: Accepts a Figma frame URL, checks whether the structure is easy for AI to understand across 4 dimensions, and outputs an improvement report.
argument-hint: <figma-url>
allowed-tools: mcp__plugin_figma_figma__get_design_context, mcp__plugin_figma_figma__get_metadata
---

# Figma AI-Readiness Audit

Analyze the Figma frame URL provided in `$ARGUMENTS` and evaluate whether it is structured in a way that allows AI to implement it accurately.

**Output language:** Respond in the same language the user is using in this conversation.

## Step 1: Parse the URL

Extract from `$ARGUMENTS`:

- `figma.com/design/:fileKey/...?node-id=X-Y` → `fileKey` and `nodeId` (convert `X-Y` to `X:Y`)

## Step 2: Fetch data

Call `get_design_context` and `get_metadata` **in parallel**:

- `get_design_context`: retrieves generated code, screenshot, annotations, and style info
- `get_metadata`: retrieves layer hierarchy and node ID structure

Parameters for `get_design_context`:
- `disableCodeConnect: true`
- `clientLanguages: "unknown"`
- `clientFrameworks: "unknown"`

## Step 3: Analyze across 4 dimensions

### Dimension A: Layer naming

**Good naming criteria:**
- Meaningful names representing section or role (e.g. `Hero`, `CardBase`, `Footer`)
- Elements inside components have role-based names (e.g. `HeroHeading`, `CTA`, `Button`)
- No duplicate names among sibling elements

**Problem patterns (checklist):**
- [ ] Auto-generated names remain (e.g. `Frame 123`, `Group 456`)
- [ ] Multiple sibling elements share the same name (e.g. multiple `CardBase` instances with identical names)
- [ ] Generic names that don't convey role (e.g. `Images`, `Text`, `Icon`)
- [ ] Text node names are the raw text content (problematic for long strings)

### Dimension B: Auto-layout usage

**Good state criteria:**
- Auto-layout is applied to vertical and horizontal groups
- Spacing is managed via padding and gap (not manually positioned)

**Problem patterns (checklist):**
- [ ] All children are absolutely positioned (no auto-layout)
- [ ] 1px-wide empty spacer frames used instead of gap settings
- [ ] Fixed px values used for spacing without tokens

### Dimension C: Variables and tokens

**Good state criteria:**
- Colors are defined as Figma variables and applied (generated code contains `var(--name)`)
- Spacing (margins, sizes) is managed with tokens
- Text styles are defined and applied (e.g. `heading/level_1`)

**Problem patterns (checklist):**
- [ ] Colors are hardcoded hex values (e.g. `#294779`) with no `var(--*)` in generated code
- [ ] Spacing uses only fixed px values (no `var(--size-*)`)
- [ ] Text styles are undefined or unapplied (no styles listed in output)

### Dimension D: Component and instance structure

**Good state criteria:**
- Repeatedly used elements are componentized
- Instances are used consistently (not manual copies)

**Problem patterns (checklist):**
- [ ] Multiple frames with similar structure exist but are not componentized
- [ ] Metadata shows `frame` instead of `instance` for repeated identical structures
- [ ] State variations are separate frames rather than component variants/props

## Step 4: Output the report

Use the following format:

---

## Figma AI-Readiness Audit Report

**File:** [file name]
**Target node:** [node name] (`nodeId`)
**Screenshot:** [display the screenshot]

---

### Overall score: X / 20

| Dimension | Score | Rating |
|-----------|-------|--------|
| A. Layer naming | X / 5 | 🟢 Good / 🟡 Room for improvement / 🔴 Needs fixing |
| B. Auto-layout | X / 5 | 〃 |
| C. Variables & tokens | X / 5 | 〃 |
| D. Component structure | X / 5 | 〃 |

Scoring criteria (5 points each):
- 5: No issues
- 3–4: Minor issues (1–2 items)
- 1–2: Needs improvement (3+ items)
- 0: Almost entirely unaddressed

---

### Strengths

(List notable practices in this file that others can learn from)

---

### Improvement suggestions

For each issue:

#### [Location / target of the issue]
- **Problem:** What is wrong
- **Impact:** How it hinders AI code generation
- **Suggestion:** Specific action to take in Figma

---

### Summary

(1–2 sentences on the overall impression and the highest-priority improvement)

---

## Error handling

- If fileKey or nodeId cannot be extracted from the URL: ask the user to provide a valid Figma frame URL
- If `get_design_context` or `get_metadata` returns an error: report the error and ask the user to check access permissions for the Figma file
- If a component URL is passed instead of a frame URL: inform the user and suggest trying with a frame-level URL
