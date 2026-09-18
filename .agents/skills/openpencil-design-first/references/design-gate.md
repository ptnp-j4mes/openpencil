# Design Gate Contract

Use this reference when handing an OpenPencil design from DESIGN MODE to IMPLEMENT MODE.

The goal is to make the approved design inspectable and reproducible without forcing the implementation agent to reverse-engineer a screenshot.

## Required artifacts

Store feature/page design artifacts together:

```text
.design/<slug>/
├── <slug>.op
└── handoff.md
```

The `.op` file is the canonical structured design. The handoff is a compact index and approval record, not a duplicate document model.

## Required handoff fields

Use this shape:

```markdown
# <Design Name>

Status: AWAITING_APPROVAL | APPROVED
Design file: .design/<slug>/<slug>.op
Page ID: <id>
Root ID: <id>

## Intent
<what user outcome this screen supports>

## Structure
<short semantic node-tree outline>

## Components
- <component/ref name> — reused | new — purpose

## Design system
- Variables/themes used: <summary>
- Style guide/preset: <name or none>
- Typography: <summary>

## Vectors/assets
- <asset/node> — path | icon_font | imported SVG | image

## Responsive behavior
- Desktop: <rule>
- Tablet: <rule>
- Mobile: <rule>

## Verification
- design_refine: PASS | findings
- lint_document: PASS | issue count and material exceptions
- visual/layout verification: PASS | method

## Implementation mapping
- <design component> -> <existing code component or TBD>

## Open questions
- <only unresolved decisions>
```

## Approval rules

`AWAITING_APPROVAL` means:
- do not edit target application code for this feature
- do not run OpenPencil codegen for this feature
- design iteration is allowed

`APPROVED` means the user explicitly approved this design revision.

When the design changes materially after approval, set the handoff back to `AWAITING_APPROVAL` and request approval for the revised design before implementation continues.

## Verification threshold

Do not block on cosmetic lint that does not affect usability or implementation. Do block handoff for:
- broken or overlapping layout
- missing required content/controls
- flattened structure that hides component semantics
- obvious token/design-system divergence
- missing responsive behavior for a responsive target
- rasterized replacements for vectors/components where structured equivalents are expected
