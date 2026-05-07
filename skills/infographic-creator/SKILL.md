---
name: infographic-creator
description: Create beautiful infographics based on given text content. Use when a user requests to create an infographic.
---

Infographics transform data, information, and knowledge into perceptible visual language. They combine visual design and data visualization, using intuitive symbols to compress complex information, helping the audience quickly understand and remember key points.

`Infographic = Information Structure + Visual Expression`

This task uses [AntV Infographic](https://infographic.antv.vision/) to create visual infographics.

Before starting the task, you need to understand the AntV Infographic syntax specification, including template lists, data structures, themes, etc.

## Language Lock

- The writing language and example language of the Prompt cannot determine the final output language.
- If the user inputs English, the generated `title`, `desc`, `label`, and other copy must remain in English; do not automatically translate them into Chinese.
- If the user inputs Chinese, the generated `title`, `desc`, `label`, and other copy must remain in Chinese; do not automatically translate them into English.

## Specifications

### AntV Infographic Syntax

AntV Infographic syntax is a custom DSL used to describe infographic rendering configurations. It uses indentation to express structure, making it suitable for AI to generate directly and output via streaming. Core information includes:

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

### Strict Syntax Rules

- The first line must be `infographic <template-name>`.
- In the template list, write only the template name itself; in the actual output, the first line must explicitly include the `infographic` prefix.
- Use `data` / `theme` blocks, with a uniform two-space indentation within the blocks.
- The key-value pair syntax is `Key Space Value`; object arrays use `-` as an item prefix.
- `icon` can use exact icon IDs, such as `mingcute/server-line`, or semantic keyword phrases, such as `star fill`.
- If using semantic keyword phrases, separate multiple keywords with spaces, not hyphens; for example, write `rocket launch`, not `rocket-launch`.
- For semantic data items under `lists`, `sequences`, `nodes`, `items`, and `compares`/`children`, `icon` should be added by default; do not omit it just because the field is optional.
- If the template name or visual style obviously depends on icons, every main data item should include an `icon`.
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
- Use `theme.base.text.font-family` to specify the font, e.g., `851tegakizatsu`.
- Use `theme.stylize` to select built-in styles and pass parameters.
  - `rough`: Hand-drawn effect
  - `pattern`: Pattern fill
  - `linear-gradient` / `radial-gradient`: Gradient style
- Output only the Infographic syntax itself, without JSON, explanatory text, or extra Markdown paragraphs.

## Data Syntax Examples

Minimal but complete positive examples are given by template category:

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

- `compare-binary-*` template

```infographic
infographic compare-binary-horizontal-simple-fold
data
  title Dining Table Price Comparison
  compares
    - label Original Price
      icon tag
      children
        - label Original Price
          value 500
          icon tag
    - label Actual Payment
      icon wallet
      children
        - label Actual Payment
          value 450
          icon check bold
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

- `chart-line-plain-text` template

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
    - id db
      label DB
  relations
    API - Read/Write -> db
```

- Fallback `items` example

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
- Comparison between two sides, solutions, or before-and-after → `compare-binary-*`
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

### Complete Output Example

```infographic
infographic list-row-horizontal-icon-arrow
data
  title Product Growth Highlights
  desc Focus on acquisition, conversion, and retention phases
  lists
    - label Acquisition
      desc Multi-channel placement and content reach
      icon rocket launch
    - label Conversion
      desc Optimize path and reduce churn
      icon chart line
    - label Retention
      desc Member benefits and tiered operations
      icon repeat
theme
  palette #3b82f6 #8b5cf6 #f97316
```

## Output Format

Output only one `plain` code block, without any explanatory text:

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
- Is `palette` a raw color value without quotes or commas
- Are swimlane nodes in `sequence-interaction-*` written as `children -> - label ...`
- Do `compare-binary-*` / `compare-hierarchy-left-right-*` have only two root nodes, with content on both sides placed in their respective `children`
- Does every item under `children` explicitly include a `label`
- Does `chart-line-plain-text` use a single ordered `values` list
- Is there no JSON, explanatory text, or extra code blocks in the output

## Generation Process

### Step 1: Understand User Requirements

Before creating an infographic, understand the user's requirements and the information they want to convey to determine the template and data structure.

If the user provides a clear content description, decompose it into a clear, concise structure.

Otherwise, clarify with the user (e.g., "Please provide a clear and concise content description.", "Which template would you like to use?")

- Extract key information structure (title, desc, items, etc.).
- Identify required data fields (title, desc, items, label, value, icon, etc.).
- Proactively add icons to main data items with clear semantics; don't wait for separate user reminders.
- Select an appropriate template.
- Describe infographic content using AntV Infographic syntax `{syntax}`.

**Key Note**: You must respect the language of the user's input. For example, if the user inputs Chinese, the text in the syntax must also be Chinese.

### Step 2: Render Infographic as standalone HTML file

After obtaining the final AntV Infographic syntax, follow these steps to generate a complete HTML file:

1. Create a complete HTML file with the following structure:
   - DOCTYPE and HTML meta (charset: utf-8)
   - Title: `{title} - Infographic`
   - Import AntV Infographic script: `https://unpkg.com/@antv/infographic@latest/dist/infographic.min.js`
   - Create a container div with id `container`
   - Initialize Infographic (`width: '100%'`, `height: '100%'`)
   - Replace `{title}` with the actual title
   - Replace `{syntax}` with the actual AntV Infographic syntax
   - Add SVG export functionality: `const svgDataUrl = await infographic.toDataURL({ type: 'svg' });`

Reference HTML template:

```html
<div id="container"></div>
<script src="https://unpkg.com/@antv/infographic@latest/dist/infographic.min.js"></script>
<script>
 const infographic = new AntVInfographic.Infographic({
    container: '#container',
    width: '100%',
    height: '100%',
  });
  infographic.render(`{syntax}`);
  document.fonts?.ready.then(() => {
    infographic.render(`{syntax}`);
  }).catch((error) => console.error('Error waiting for fonts to load:', error));
</script>
```

2. Use the Write tool to generate the HTML file, naming it `<title>-infographic.html`

3. Present to the user:
   - Provide the generated file path and prompt: "Open directly in a browser to view and save the SVG"
   - Output the syntax and prompt: "Let me know if you need to adjust the template/palette/content"

**Note:** The HTML file must include:

- SVG export via an export button
- Responsive container with 100% width and height

### Alternative Step 2: Use AntV Infographic syntax for embedding in Open-Slides pages

If the user wants to embed the infographic in an Open-Slides page, use the following template:

```ts
 const infographic = new AntVInfographic.Infographic({
    container: '#container',
    width: '100%',
    height: '100%',
  });
  infographic.render(`{syntax}`);
  document.fonts?.ready.then(() => {
    infographic.render(`{syntax}`);
  }).catch((error) => console.error('Error waiting for fonts to load:', error));

```