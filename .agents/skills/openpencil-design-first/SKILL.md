---
name: openpencil-design-first
description: Use when designing or redesigning a web/app UI before implementation with OpenPencil MCP, or when the task requires a structured design artifact such as a node tree, reusable components, variables/tokens, vectors, auto-layout, responsive rules, and an approval gate. Do not use for code-only fixes that do not change the product design.
---

# OpenPencil Design-First

Produce an inspectable OpenPencil design before frontend implementation. Treat the canonical `.op` document, its node tree, components, variables, vectors, and exported design contract as the design source of truth.

## Non-negotiable design gate

For new UI, redesigned UI, or a material layout change:

1. Enter **DESIGN MODE**.
2. Do not edit application source code.
3. Do not call `codegen_plan`, `codegen_submit_chunk`, `codegen_assemble`, or other implementation/code-generation steps.
4. Build and verify the design in OpenPencil.
5. Present the design handoff and stop.
6. Enter **IMPLEMENT MODE** only after the user explicitly approves the current design.

An explicit approval can be wording such as `approved`, `approve design`, `อนุมัติ design`, or an equally clear instruction to implement the reviewed design. Do not infer approval from silence.

If the user explicitly says to skip design and implement directly, follow that instruction.

## DESIGN MODE workflow

### 1. Compile design context

Read the task requirements and the target repository's existing UI conventions before drawing.

Prefer existing:
- design tokens and variables
- component names and semantics
- page-shell/layout patterns
- typography and spacing conventions
- domain terminology
- responsive conventions

Do not invent a second design system when a project one already exists.

### 2. Load the OpenPencil design prompt

Call `get_design_agent_prompt` with the user's design request.

Use `verifyProtocol="screenshot"` when a renderer is available. Use `verifyProtocol="layout"` when screenshot verification is unavailable.

Load only additional style-guide/prompt segments needed for the task.

### 3. Establish design system inputs

Inspect existing OpenPencil variables with `get_variables`.

When appropriate, use:
- `get_style_guide_tags` / `get_style_guide`
- `apply_design_system`
- `set_variables` / `set_themes`

Prefer semantic variable references over duplicated literal values.

### 4. Build structure before polish

Use the layered MCP workflow:

`design_skeleton` → `design_content` → `design_refine`

The design must use semantic, inspectable nodes. Do not flatten a whole screen into one image.

For reusable patterns:
- create real reusable components with `create_component`
- instantiate them with `instantiate_component`
- keep component names stable and implementation-friendly

For vector assets:
- use path/vector/icon nodes or SVG import
- preserve vector structure when practical
- do not replace vectors with raster screenshots merely to make the preview look correct

### 5. Verify the design

Run `design_refine` for the target root and inspect its `layoutSnapshot`.

Run `lint_document` for the target subtree. Fix material lint issues before handoff.

Inspect the final tree with `batch_get` at sufficient depth to verify:
- hierarchy
- naming
- component/ref usage
- layout structure
- interactive-control node types
- important visual properties

Inspect `get_variables` again to verify token usage.

### 6. Persist the design contract

Use `set_design_md` / `export_design_md` so the `.op` document carries its design specification.

Save the design with `save_document` under the target project's design workspace when one exists. Recommended path:

`.design/<feature-or-page>/<feature-or-page>.op`

Create a handoff file next to it:

`.design/<feature-or-page>/handoff.md`

Follow [references/design-gate.md](references/design-gate.md) for the required handoff content.

### 7. Stop at the gate

Report:
- design file path
- page/root id
- key components
- token/design-system usage
- lint result
- responsive assumptions
- unresolved design questions, if any

Set the conversational state to **AWAITING DESIGN APPROVAL** and stop. Do not implement application code in the same step.

## IMPLEMENT MODE workflow

Start only after explicit approval.

1. Re-read the approved `.op` design and handoff.
2. Read the target codebase's real components before creating new ones.
3. Map design components to existing implementation components whenever semantics match.
4. Preserve design-system tokens instead of copying hard-coded visual values.
5. Use `codegen_plan` only if code generation is useful; generated code is a starting point, not authority over the target repository's architecture.
6. Implement in the target stack.
7. Render the implementation.
8. Compare implementation against the approved design for layout, component structure, content, and responsive behavior.
9. Fix meaningful drift before handoff.

## Scope rules

Use this workflow for:
- new pages
- page redesigns
- new dashboards/forms/tables
- material component redesign
- design-system or navigation changes
- tasks explicitly requesting node tree/component/vector design

Skip the design gate for:
- backend-only changes
- tests
- data/migration work
- refactors with no UI change
- tiny UI bug fixes where the desired visual result is already unambiguous
