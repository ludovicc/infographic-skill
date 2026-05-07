# Infographic Syntax Generation Specification

This document is used to guide the generation of plain text output that complies with the AntV Infographic syntax specification.

## Table of Contents

- Objectives and Input/Output
- Syntax Structure
- Syntax Specification
- Syntax Examples
- Available Templates
- Template Selection Suggestions
- Generation Process
- Output Format
- Self-Checklist

## Objectives and Input/Output

- **Input**: User's text content or requirement description
- **Output**: A single fenced code block containing only Infographic syntax
- **Responsibility Boundary**: This specification is only responsible for generating syntax, not for generating HTML files, React/TSX components, or template source code.

## Syntax Structure

Infographic syntax consists of an entry and block structures:

- **Entry**: `infographic <template-name>`
- **Block**: `data` / `theme`
- **Indentation**: Use two spaces consistently for levels within a block.

## Syntax Specification

### AntV Infographic Syntax

AntV Infographic syntax is a custom YAML-based DSL used to describe infographic rendering configurations. It uses indentation to express structure, making it suitable for AI to generate directly and output via streaming. Core information includes:

1. template: Express information structure using templates.
2. data: Infographic data, including `title`, `desc`, and main data fields.
3. theme: Theme configuration, including `palette`, fonts, stylization, etc.

Example:

```infographic
infographic list-row-horizontal-icon-arrow
data
  title Title
  desc Description
  lists
    - label Label
      value 12.5
      desc Explanation
      icon document text
theme
  palette #3b82f6 #8b5cf6 #f97316
```

### Syntax Rules

- The first line must be `infographic <template-name>`.
- Use `data` / `theme` blocks.
- Key-value pair syntax is `Key Space Value`; object arrays use `-` as an item prefix and include a new line.
- `title`, `desc`, `label`, and other reader-facing copy should remain consistent with the user's input language by default; do not translate into another language unless explicitly requested.
- If the user input is a mix of languages, prioritize keeping original proper nouns, product names, and abbreviations unchanged, and have the supplementary copy follow the primary input language.
- `icon` can use exact icon IDs, such as `mingcute/server-line`, or semantic keyword phrases, such as `star fill`.
- If using semantic keyword phrases, separate multiple keywords with spaces, not hyphens; for example, write `rocket launch`, not `rocket-launch`.
- For semantic data items under `lists`, `sequences`, `nodes`, `items`, and `compares`/`children`, `icon` should be added by default; do not omit it just because the field is optional.
- If the template name or visual style obviously depends on icons (e.g., the name contains "icon", or the template itself is an icon-based card/node), every main data item should include an `icon`.
- If unsure of the exact icon ID, prioritize writing a concise semantic keyword, such as `rocket`, `shield`, `database`, `users`, `chart line`, rather than omitting the `icon` field.
- Omit the `icon` only for pure chart-like data points, pure numerical sequences, or when the user explicitly requests a minimalist text version.
- Try to use pure numerical values for `value`; numerical units should prioritize being expressed in `label` or `desc`.
- `palette` is recommended to use an inline simple array format, for example: `palette #4f46e5 #06b6d4 #10b981`.
- Color values in `palette` are raw values, without quotes or commas.
- `data` should contain only one main data field that matches the template, avoiding mixed use of `lists`, `sequences`, `compares`, `values`, `root`, `nodes`.

Main Data Field Selection Rules:

- `list-*` → `lists`
- `sequence-*` → `sequences`, optional `order asc|desc`
- `sequence-interaction-*` → `sequences` + `relations`
  - `sequences` represents a list of swimlanes
  - Each swimlane must include a `label`
  - The `children` of each swimlane represents a list of nodes
  - Each item under `children` must be written as an object entry and include a `label`
  - Nodes can optionally include `id`, `icon`, `step`, `desc`, `value`
  - `step` is used to represent time levels; items with the same `step` are at the same height
- `compare-*` → `compares`
  - `compare-binary-*` / `compare-hierarchy-left-right-*`
    - The first level of `compares` must and can only have two root nodes, representing the two sides of the comparison.
    - Both root nodes should include `children`.
    - Actual comparison items are written under their respective `children`.
    - Each item under `children` must be written as an object entry and include a `label`.
    - Even if there is only 1 metric on each side, it should be written as 1 object entry within `children`.
  - `compare-swot`
    - `compares` can directly contain multiple root nodes.
    - Each root node can optionally include `children`.
  - `compare-quadrant-*`
    - `compares` directly contains 4 quadrant root nodes.
- `hierarchy-structure` → `items`
- `hierarchy-*` → Single `root`, recursively nested via `children`.
- `relation-*` → `nodes` + `relations`
  - Simple relations can also be expressed directly using arrow syntax.
- `chart-*` → `values`
  - `chart-line-plain-text` / `chart-bar-plain-text` / `chart-column-simple` all use a single ordered `values` list.
  - Each data point uses `label` for the category and `value` for the numerical value.
  - The order of the line chart is expressed by the arrangement order of items in `values`.
- Use `items` as a fallback when the structure cannot be clearly determined.

Theme Rules:

- `theme` is used for custom themes, such as `palette`, `base`, `stylize`.
- Use `theme.base.text.font-family` to specify the global font, e.g., `851tegakizatsu`, `AliPuHuiTi`.
- Use `theme.stylize` to select built-in styles and pass parameters.
  - `rough`: Hand-drawn effect
  - `pattern`: Pattern fill
  - `linear-gradient` / `radial-gradient`: Gradient style
- Output only the Infographic syntax itself, without JSON, explanatory text, or extra Markdown paragraphs.

## Data Syntax Examples

- `list-*` template

```infographic
infographic list-grid-badge-card
data
  title Feature List
  lists
    - label Fast
      icon flash fast
    - label Secure
      icon shield check
```

- `sequence-*` template

```infographic
infographic sequence-ascending-steps
data
  title Release Process
  sequences
    - label Requirement Confirmation
      icon clipboard check
    - label Development Implementation
      icon code
    - label Release Live
      icon rocket
  order asc
```

- `sequence-interaction-*` template

```infographic
infographic sequence-interaction-compact-animated-badge-card
data
  title Login Verification Process
  sequences
    - label User
      icon user
      children
        - label Initiate Login
          id user-login
          step 0
          icon login
        - label Receive Result
          id user-result
          step 2
          icon inbox check
    - label Server
      icon server
      children
        - label Verify Credentials
          id server-verify
          step 1
          icon shield check
        - label Return Result
          id server-return
          step 2
          icon send
  relations
    user-login - Submit Account/Password -> server-verify
    server-verify - Generate Result -> server-return
    server-return - Return Result -> user-result
```

- `hierarchy-*` template

```infographic
infographic hierarchy-tree-curved-line-rounded-rect-node
data
  title Organizational Structure
  root
    label Company
    children
      - label Product Dept
      - label Tech Dept
```

- `compare-swot` template

```infographic
infographic compare-swot
data
  title Product SWOT
  compares
    - label Strengths
      icon trophy
      children
        - label High Brand Awareness
          icon star
    - label Weaknesses
      icon alert circle
      children
        - label High Cost Pressure
          icon wallet
```

- `compare-quadrant-*` template

```infographic
infographic compare-quadrant-quarter-simple-card
data
  title Task Priority
  compares
    - label High Value, Low Cost
    - label High Value, High Cost
    - label Low Value, Low Cost
    - label Low Value, High Cost
```

- `compare-*` template

Use `children` to form a comparison hierarchy, where each root node represents a comparison object, and child nodes represent specific metrics for that object, for example:

```infographic
infographic compare-binary-horizontal-simple-fold
data
  title Annual Sale Promotion
  compares
    - label Weekdays
      icon calendar
      children
        - label Original Price 500
          icon tag
        - label Up to 10% Off
          icon percent
    - label Big Sale
      icon megaphone
      children
        - label Actual Payment 450
          icon wallet
        - label Up to 20% Off
          icon badge percent
```

- `chart-*` template

Flatten each data item under `values`, for example:

```infographic
infographic chart-line-plain-text
data
  title Model A Accuracy Change
  desc Most significant improvement in Week 4
  values
    - label Week1
      value 86.5
    - label Week2
      value 87.3
    - label Week3
      value 89.1
    - label Week4
      value 91.2
theme
  palette #4f46e5 #db2777 #14b8a6
```

- `relation-*` template

```infographic
infographic relation-dagre-flow-tb-simple-circle-node
data
  title System Relations
  nodes
    - label API
      icon api
    - id db
      label DB
      icon database
  relations
    API - Read/Write -> db
```

- Fallback `items` Example

If it's unclear which data field to use, or if the information structure is relatively simple, you can use the `items` list directly, for example:

```infographic
infographic list-row-horizontal-icon-arrow
data
  title Summary of Key Points
  items
    - label Efficiency First
      desc Focus on key actions
    - label Result Oriented
      desc Output actionable conclusions
```

### Available Templates

Listed below are common template names, not the exhaustive set; in the actual output, the first line must be `infographic <template-name>`:

- chart-bar-plain-text
- chart-column-simple
- chart-line-plain-text
- chart-pie-compact-card
- chart-pie-donut-pill-badge
- chart-pie-donut-plain-text
- chart-pie-plain-text
- chart-wordcloud
- compare-binary-horizontal-badge-card-arrow
- compare-binary-horizontal-simple-fold
- compare-binary-horizontal-underline-text-vs
- compare-hierarchy-left-right-circle-node-pill-badge
- compare-quadrant-quarter-circular
- compare-quadrant-quarter-simple-card
- compare-swot
- hierarchy-mindmap-branch-gradient-capsule-item
- hierarchy-mindmap-level-gradient-compact-card
- hierarchy-structure
- hierarchy-tree-curved-line-rounded-rect-node
- hierarchy-tree-tech-style-badge-card
- hierarchy-tree-tech-style-capsule-item
- list-column-done-list
- list-column-simple-vertical-arrow
- list-column-vertical-icon-arrow
- list-grid-badge-card
- list-grid-candy-card-lite
- list-grid-ribbon-card
- list-row-horizontal-icon-arrow
- list-sector-plain-text
- list-waterfall-badge-card
- list-waterfall-compact-card
- list-zigzag-down-compact-card
- list-zigzag-down-simple
- list-zigzag-up-compact-card
- list-zigzag-up-simple
- relation-dagre-flow-tb-animated-badge-card
- relation-dagre-flow-tb-animated-simple-circle-node
- relation-dagre-flow-tb-badge-card
- relation-dagre-flow-tb-simple-circle-node
- sequence-ascending-stairs-3d-underline-text
- sequence-ascending-steps
- sequence-circular-simple
- sequence-color-snake-steps-horizontal-icon-line
- sequence-cylinders-3d-simple
- sequence-filter-mesh-simple
- sequence-funnel-simple
- sequence-horizontal-zigzag-underline-text
- sequence-mountain-underline-text
- sequence-pyramid-simple
- sequence-roadmap-vertical-plain-text
- sequence-roadmap-vertical-simple
- sequence-snake-steps-compact-card
- sequence-snake-steps-simple
- sequence-snake-steps-underline-text
- sequence-stairs-front-compact-card
- sequence-stairs-front-pill-badge
- sequence-timeline-rounded-rect-node
- sequence-timeline-simple
- sequence-zigzag-pucks-3d-simple
- sequence-zigzag-steps-underline-text
- sequence-interaction-default-badge-card
- sequence-interaction-default-animated-badge-card
- sequence-interaction-default-compact-card
- sequence-interaction-default-capsule-item
- sequence-interaction-default-rounded-rect-node

## Template Selection Suggestions

- Strict order, step advancement, stage evolution → `sequence-*`
- Multi-role or multi-system interaction → `sequence-interaction-*`
- List of parallel key points → `list-row-*` / `list-column-*` / `list-grid-*`
- Comparison between two sides, solutions, or before-and-after → `compare-*`
  - First determine who the two sides are
  - Then expand `children` for both sides respectively
- SWOT Analysis → `compare-swot`
- Quadrant Analysis → `compare-quadrant-*`
- Hierarchical tree structure → `hierarchy-tree-*`
- Statistical trends, single sequence changes → `chart-line-plain-text`
- Statistical comparison, comparison of single group values → `chart-bar-plain-text` / `chart-column-simple`
- Node relationships, process dependencies → `relation-*`
- Word cloud theme display → `chart-wordcloud`
- Mind map → `hierarchy-mindmap-*`

## Generation Process

1. Extract title, description, items, sequence, and hierarchical relationships from the user's content.
2. If the user explicitly specifies a template name and that template is compatible with the content structure, prioritize using that template.
3. If the user only specifies a template series, choose the best matching template within that series.
4. Otherwise, determine the information structure and select a matching template series.
5. Organize `data`, keeping only the main data field corresponding to the template.
6. Proactively add icons to main data items with clear semantics; if the template itself is icon-based, do not omit them by default.
7. Keep `title`, `desc`, `label`, and other copy consistent with the user's input language; only perform translation or bilingualization when explicitly requested.
8. Supplement with `theme` when the user specifies style, palette, or font.
9. Output a single code block containing only Infographic syntax.

## Output Format

Output only one code block, without any explanatory text:

```infographic
infographic list-row-horizontal-icon-arrow
data
  title Title
  desc Description
  lists
    - label Item
      value 12.5
      desc Explanation
      icon document text
theme
  palette #3b82f6 #8b5cf6 #f97316
```

## Self-Checklist

Check the following items before outputting:

- Is the first line `infographic <template-name>`
- Was only one main data field matching the template used
- Do main data items have reasonable icons, especially for lists, steps, nodes, comparison items, and templates with "icon" in their names
- Is the copy language consistent with the user's input language, without unnecessary translation
- Is `palette` a raw color value without quotes or commas
- Are swimlane nodes in `sequence-interaction-*` written as `children -> - label ...`
- Do `compare-binary-*` / `compare-hierarchy-left-right-*` have only two root nodes, with content on both sides placed in their respective `children`
- Does every item under `children` explicitly include a `label`
- Does `chart-line-plain-text` use a single ordered `values` list
- Is there no JSON, explanatory text, or extra code blocks in the output
