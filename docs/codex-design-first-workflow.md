# Codex Design-First Workflow

This repository already contains the design engine needed for a design-first Codex flow. The canonical `.op` model lives in `op-editor-core`, MCP design operations live in `op-mcp`, and code generators consume the canonical document.

The design-first layer added in this fork intentionally stays thin: it coordinates the existing MCP tools and adds a human approval gate instead of introducing a second node model or a second codegen pipeline.

## Architecture

```text
Requirement / business rule
          |
          v
        Codex
          |
          v
openpencil-design-first skill
          |
          v
      OpenPencil MCP
          |
          +--> design_skeleton
          +--> design_content
          +--> components / variables / vectors
          +--> design_refine
          +--> lint_document
          +--> batch_get / get_variables
          +--> export_design_md
          +--> save_document
          |
          v
       .op design
          |
          v
   AWAITING APPROVAL
          |
     explicit approval
          |
          v
    implementation/codegen
```

## Why the gate is outside the core document model

OpenPencil already has one canonical structured document: `PenDocument`. Adding another persisted design graph would create drift.

The approval state belongs to the product-development workflow rather than rendering semantics. The skill therefore records approval in the target repository's `handoff.md` and uses the existing `.op` document for the actual design.

## Existing MCP capabilities used

The workflow builds on existing tools rather than replacing them:

- `get_design_agent_prompt` for task-matched design guidance
- `design_skeleton` / `design_content` / `design_refine` for layered design generation
- component tools for reusable nodes and instances
- variable/theme tools for design tokens
- vector/path/SVG-capable node operations
- `lint_document` for structural/design checks
- `batch_get` for node-tree inspection
- `get_variables` for token inspection
- `set_design_md` / `export_design_md` for the embedded design specification
- `save_document` for the canonical `.op` artifact
- `codegen_plan` and related tools only after approval

## Codex skill location

The repository skill is stored at:

```text
.agents/skills/openpencil-design-first/SKILL.md
```

Codex scans repository `.agents/skills` directories. To use the same workflow in another application repository, copy the skill directory into that repository's `.agents/skills/` directory and configure that Codex environment to reach the OpenPencil MCP server.

## Recommended target-repository layout

```text
target-app/
├── .agents/
│   └── skills/
│       └── openpencil-design-first/
└── .design/
    └── <feature>/
        ├── <feature>.op
        └── handoff.md
```

Do not commit generated screenshots as the source of truth. They may be kept as review evidence, but the `.op` document remains the structured source.
