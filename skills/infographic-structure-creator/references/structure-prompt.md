# Infographic Structure Component Generation Specification

This document is used to guide the generation of Structure component code that complies with the framework specifications.

## Table of Contents

- Core Framework Concepts
- Structure Classification System
- Technical Specifications
- Code Generation Requirements
- Generation Process
- Reference Examples
- Output Format

## Core Framework Concepts

The infographic framework consists of three core parts:

- **Structure**: Responsible for the overall layout and organization of data items.
- **Title**: Optional title component.
- **Item/Items**: Component for displaying single or multiple information units.

Structure is the entry-point component. By combining Title and Item/Items, along with layout logic and interaction buttons, a complete infographic is formed. For hierarchical structures, the `Items` array can be used to pass multiple components (such as root and child node components).

## Structure Classification System

Based on information organization characteristics, structures are divided into the following types:

1. **List Structure (list-*)**: Information items arranged side by side, without clear directionality or hierarchical relationships.
   - Horizontal list, vertical list, grid list, waterfall flow, etc.

2. **Comparison Structure (compare-*)**: Clear binary or multi-party comparison layouts.
   - Left-right comparison, top-bottom comparison, multi-item comparison, mirror comparison, etc.

3. **Sequence Structure (sequence-*)**: Information flow with clear directionality and sequence.
   - Timeline, step process, staircase, S-shaped flow, etc.

4. **Hierarchy Structure (hierarchy-*)**: Tree-like, nested, or layouts with clear primary-secondary relationships.
   - Tree shape, pyramid, radial, nested circles, etc.

5. **Relation Structure (relation-*)**: Displays connections, dependencies, or interactions between elements.
   - Network diagrams, matrices, cycle diagrams, Venn diagrams, etc.

6. **Geographic Structure (geo-*)**: Information organization based on geographic space or physical location.
   - Map annotations, regional distributions, roadmaps, etc.

7. **Statistical Chart (chart-*)**: Displays quantitative data relationships in chart form.
   - Bar chart, pie chart, line chart, radar chart, etc.

## Technical Specifications

### 1. Type Definitions

```tsx
export interface BaseStructureProps {
  Title?: ComponentType<Pick<TitleProps, 'title' | 'desc'>>;
  Item?: ComponentType<
    Omit<BaseItemProps, 'themeColors'> &
      Partial<Pick<BaseItemProps, 'themeColors'>>
  >;
  Items?: ComponentType<Omit<BaseItemProps, 'themeColors'>>[];
  data: Data;
  options: ParsedInfographicOptions;
}

export interface Data {
  title?: string;
  desc?: string;
  items: ItemDatum[];
  illus?: Record<string, string | ResourceConfig>;
  [key: string]: any;
}

export interface ItemDatum {
  icon?: string | ResourceConfig;
  label?: string;
  desc?: string;
  value?: number;
  illus?: string | ResourceConfig;
  children?: ItemDatum[];
  [key: string]: any;
}

export interface BaseItemProps {
  x?: number;
  y?: number;
  id?: string;
  indexes: number[];
  data: Data;
  datum: Data['items'][number];
  themeColors?: ThemeColors;
  positionH?: 'normal' | 'center' | 'flipped';
  positionV?: 'normal' | 'middle' | 'flipped';
  width?: number;
  height?: number;
  [key: string]: any;
}
```

**IMPORTANT:**

- For simple structures, use the `Item` attribute to pass a single component.
- For hierarchical structures (e.g., trees, pyramids), use the `Items` array to pass multiple components; different levels can use different component styles.
- `options` contains theme configurations, palettes, etc., and can be accessed via utility functions.
- `themeColors` is optional in `BaseItemProps`; some components will pass it in custom.

### 2. Available Components List

**MUST select from the following components; do not use components not listed:**

#### Atomic Components (Imported from ../../jsx)

All atomic components uniformly use `x`, `y`, `width`, `height` attributes to define position and size; do not use native SVG attributes like `cx/cy/r`.

- **Defs**: Defines SVG definitions such as gradients and filters.

  ```tsx
  <Defs>{/* Definitions for gradients, filters, etc. */}</Defs>
  ```

- **Ellipse**: Ellipse shape

  ```tsx
  <Ellipse x={0} y={0} width={100} height={60} fill="blue" />
  // NOTE:
  // 1. x/y are the top-left position, not the center point.
  // 2. Use width/height, not rx/ry.
  // 3. When drawing a circle, width and height are equal.
  ```

- **Group**: Grouping container

  ```tsx
  <Group x={10} y={10}>
    {children}
  </Group>
  ```

- **Path**: Path shape

  ```tsx
  <Path
    d="M 0 0 L 100 100"
    stroke="black"
    strokeWidth={2}
    width={100}
    height={100}
  />
  // width/height is the estimated size of d.
  ```

- **Rect**: Rectangle shape

  ```tsx
  <Rect x={0} y={0} width={100} height={50} fill="red" />
  ```

- **Text**: Text element (supports line wrapping)

  ```tsx
  <Text
    x={0}
    y={0}
    width={100}
    height={50}
    fontSize={14}
    fontWeight="normal" // or 'bold'
    alignHorizontal="center" // 'left' | 'center' | 'right'
    alignVertical="middle" // 'top' | 'middle' | 'bottom'
    fill="#000000"
  >
    Text Content
  </Text>
  // NOTE: Text content is passed as children, not as a text attribute.
  ```

- **Polygon**: Polygon
  ```tsx
  <Polygon
    points={[
      { x: 0, y: 0 },
      { x: 100, y: 0 },
      { x: 50, y: 100 },
    ]}
    fill="green"
  />
  // NOTE: points is an object array {x, y}[], not a string.
  ```

#### Encapsulated Components (Imported from ../components)

- **BtnAdd**: Add button, requires `indexes` attribute.

  ```tsx
  <BtnAdd indexes={[0]} x={10} y={20} />
  ```

- **BtnRemove**: Remove button, requires `indexes` attribute.

  ```tsx
  <BtnRemove indexes={[0]} x={10} y={20} />
  ```

- **BtnsGroup**: Button group container.

  ```tsx
  <BtnsGroup>{btnElements}</BtnsGroup>
  ```

- **ShapesGroup**

Attributes and usage are identical to Group, but internal shapes can be stylized.

```tsx
<ShapesGroup>
  <Rect width={100} height={100} />
  <Rect x={100} width={100} height={100} />
  <Rect x={200} width={100} height={100} />
</ShapesGroup>
```

- **ItemsGroup**: Data items group container.

  ```tsx
  <ItemsGroup>{itemElements}</ItemsGroup>
  ```

- **Illus**: Illustration component (replaced by image or SVG).

  ```tsx
  <Illus x={0} y={0} width={200} height={150} />
  ```

- **Title**: Default title component.

  ```tsx
  <Title title="Title" desc="Description" alignHorizontal="center" />
  ```

- **ItemLabel**: Data item label.

  ```tsx
  <ItemLabel indexes={[0]} x={0} y={0}>
    Label
  </ItemLabel>
  ```

- **ItemDesc**: Data item description.

  ```tsx
  <ItemDesc indexes={[0]} x={0} y={0}>
    Description
  </ItemDesc>
  ```

- **ItemIcon**: Data item icon.

  ```tsx
  <ItemIcon indexes={[0]} x={0} y={0} size={40} />
  ```

- **ItemValue**: Data item value.

  ```tsx
  <ItemValue indexes={[0]} value={100} x={0} y={0} />
  ```

- **ItemIconCircle**: Circular icon component.
  ```tsx
  <ItemIconCircle indexes={[0]} x={0} y={0} size={50} fill="#000000" />
  ```

#### Decoration Components (Imported from ../decorations)

- **SimpleArrow**: Simple arrow decoration.

  ```tsx
  <SimpleArrow
    x={0}
    y={0}
    width={25}
    height={25}
    colorPrimary="#000000"
    rotation={0} // Optional, rotation angle: 0, 90, 180, 270
  />
  ```

- **Triangle**: Triangle decoration.
  ```tsx
  <Triangle
    x={0}
    y={0}
    width={10}
    height={8}
    rotation={0}
    colorPrimary="#000000"
  />
  ```

#### Definition Components (Imported from ../defs)

- **LinearGradient**: Linear gradient definition.
  ```tsx
  <Defs>
    <LinearGradient
      id="my-gradient"
      startColor="#ff0000"
      stopColor="#0000ff"
      direction="left-right" // 'left-right' | 'right-left' | 'top-bottom' | 'bottom-top'
    />
  </Defs>
  <Rect fill="url(#my-gradient)" />
  ```

**Using Native SVG Elements in Defs:**
Native SVG elements can be used inside the `<Defs>` tag:

```tsx
<Defs>
  <linearGradient
    id="gradient-id"
    x1="0%"
    y1="0%"
    x2="100%"
    y2="100%"
    gradientUnits="userSpaceOnUse"
  >
    <stop offset="0%" stopColor="#ff0000" />
    <stop offset="100%" stopColor="#0000ff" />
  </linearGradient>
</Defs>
```

#### Layout Components (Imported from ../layouts)

- **FlexLayout**: Flexbox layout.
  ```tsx
  <FlexLayout
    flexDirection="row" // 'row' | 'column' | 'row-reverse' | 'column-reverse'
    justifyContent="center" // 'flex-start' | 'flex-end' | 'center' | 'space-between'
    alignItems="center" // 'flex-start' | 'flex-end' | 'center'
    alignContent="center" // 'flex-start' | 'flex-end' | 'center' | 'space-between'
    flexWrap="wrap" // 'wrap' | 'nowrap'
    gap={20}
  >
    {children}
  </FlexLayout>
  ```

#### Stylized Rendering

Stylized rendering refers to transforming shapes into stylized graphics during the rendering stage, such as a hand-drawn style.

Shapes identified in the following ways can be stylized (implemented by the renderer):

1. Add `data-element-type="shape"` attribute

```tsx
<Rect data-element-type="shape" width="100" height="100" />
```

2. Wrap with ShapesGroup

```tsx
<ShapesGroup>
  <Rect width="100" height="100" />
  <Rect x="100" width="100" height="100" />
  <Rect x="200" width="100" height="100" />
</ShapesGroup>
```

> Stylized rendering only supports graphical elements (such as Path, Ellipse, Rect, Polygon, etc.), not text elements or groups.

#### Utility Functions

**Layout Calculation Functions (Imported from ../../jsx):**

- **getElementBounds**: Get element boundary information.
  ```tsx
  const bounds = getElementBounds(<Rect width={100} height={50} />);
  // Returns: { x: number, y: number, width: number, height: number }
  ```

**Theme and Color Functions (Imported from ../utils):**

- **getPaletteColor**: Get the color at a specified index in the palette.

  ```tsx
  const color = getPaletteColor(options, [index]); // Returns a color string
  ```

- **getPaletteColors**: Get the complete array of palette colors.

  ```tsx
  const palette = getPaletteColors(options); // Returns an array of colors
  ```

- **getColorPrimary**: Get the primary theme color.

  ```tsx
  const colorPrimary = getColorPrimary(options); // Returns the primary color string
  ```

- **getThemeColors**: Get the theme configuration.
  ```tsx
  const themeColors = getThemeColors(options.themeConfig);
  // Or custom configuration
  const themeColors = getThemeColors(
    {
      colorPrimary: '#FF356A',
      colorBg: '#ffffff',
    },
    options,
  );
  // Returns a theme object containing colorText, colorPrimaryBg, etc.
  ```

**Data Processing Functions (Imported from ../../utils):**

- **getDatumByIndexes**: Get a data item based on indexes.
  ```tsx
  const datum = getDatumByIndexes(items, [0, 1]); // Get nested data
  ```

**Component Selection Functions (Imported from ../utils):**

- **getItemComponent**: Get the Item component for a specified level (used for the Items array).
  ```tsx
  const ItemComponent = getItemComponent(Items, level);
  // Items is an array of components, level is the hierarchy index
  // If level exceeds the array length, returns the last component
  ```

### 3. On-demand Import

```tsx
import type { ComponentType, JSXElement } from '../../jsx';
import {
  getElementBounds,
  Defs,
  Ellipse,
  Group,
  Path,
  Polygon,
  Rect,
  Text,
} from '../../jsx';
import {
  BtnAdd,
  BtnRemove,
  BtnsGroup,
  Illus,
  ItemDesc,
  ItemIcon,
  ItemIconCircle,
  ItemLabel,
  ItemsGroup,
  ItemValue,
  Title,
} from '../components';
import { LinearGradient } from '../defs';
import { SimpleArrow, Triangle } from '../decorations';
import { FlexLayout } from '../layouts';
import {
  getColorPrimary,
  getPaletteColor,
  getPaletteColors,
  getThemeColors,
  getItemComponent,
} from '../utils';
import { getDatumByIndexes } from '../../utils';
import { registerStructure } from './registry';
import type { BaseStructureProps } from './types';
```

**Notes:**

- Only import components and functions that are actually used.
- For hierarchical structures, remember to import the `BaseItemProps` type (if needed).
- Import decoration and definition components as needed.

Supported Third-Party Libraries:

- **d3**: Used for complex layout calculations such as force-directed and hierarchical layouts.
- **lodash-es**: General-purpose utility functions.
- **tinycolor2**: Color handling.

> Other libraries can be introduced based on actual needs.

### 4. Component Structure Template

**Simple Structure Template (Using Item):**

```tsx
export interface [StructureName]Props extends BaseStructureProps {
  gap?: number;
  // Custom parameters in addition to BaseStructureProps
}

export const [StructureName]: ComponentType<[StructureName]Props> = (props) => {
  const { Title, Item, data, gap = 20, options } = props;
  const { title, desc, items = [] } = data;

  // 1. Handle Title
  const titleContent = Title ? <Title title={title} desc={desc} /> : null;

  // 2. Get Element Sizes
  const btnBounds = getElementBounds(<BtnAdd indexes={[0]} />);
  const itemBounds = getElementBounds(
    <Item indexes={[0]} data={data} datum={items[0]} />
  );

  // 3. Prepare Element Arrays
  const btnElements: JSXElement[] = [];
  const itemElements: JSXElement[] = [];
  const decorElements: JSXElement[] = []; // Decoration elements (such as arrows, lines, etc.)

  // 4. Iterate Data Items to Generate Elements
  items.forEach((item, index) => {
    const indexes = [index];

    // Calculate position and add Item
    itemElements.push(
      <Item
        indexes={indexes}
        datum={item}
        data={data}
        x={/* Calculate x */}
        y={/* Calculate y */}
      />
    );

    // Add remove button
    btnElements.push(
      <BtnRemove
        indexes={indexes}
        x={/* Calculate x */}
        y={/* Calculate y */}
      />
    );

    // Add insert button
    btnElements.push(
      <BtnAdd
        indexes={indexes}
        x={/* Calculate x */}
        y={/* Calculate y */}
      />
    );
  });

  // 5. Add Final Add Button
  if (items.length > 0) {
    btnElements.push(
      <BtnAdd
        indexes={[items.length]}
        x={/* Calculate x */}
        y={/* Calculate y */}
      />
    );
  }

  // 6. Return Layout
  return (
    <FlexLayout
      id="infographic-container"
      flexDirection="column"
      justifyContent="center"
      alignItems="center"
    >
      {titleContent}
      <Group>
        <Group>{decorElements}</Group>
        <ItemsGroup>{itemElements}</ItemsGroup>
        <BtnsGroup>{btnElements}</BtnsGroup>
      </Group>
    </FlexLayout>
  );
};

registerStructure('[structure-name]', {
  component: [StructureName],
  composites: ['title', 'item'], // Fill in based on the components actually used.
});
```

**Hierarchical Structure Template (Using Items array):**

```tsx
export interface [StructureName]Props extends BaseStructureProps {
  gap?: number;
  // Custom parameters in addition to BaseStructureProps
}

export const [StructureName]: ComponentType<[StructureName]Props> = (props) => {
  const { Title, Items, data, gap = 20, options } = props;
  const [RootItem, ChildItem] = Items; // Destructure to get components at different levels.
  const { title, desc, items = [] } = data;

  const titleContent = Title ? <Title title={title} desc={desc} /> : null;

  const btnElements: JSXElement[] = [];
  const itemElements: JSXElement[] = [];

  // Get dimensions for root and child nodes.
  const rootItemBounds = getElementBounds(
    <RootItem indexes={[0]} data={data} datum={items[0]} />
  );
  const childItemBounds = getElementBounds(
    <ChildItem indexes={[0, 0]} data={data} datum={items[0]?.children?.[0] || {}} />
  );

  // Iterate root nodes
  items.forEach((rootItem, rootIndex) => {
    const { children = [] } = rootItem;

    // Render root node
    itemElements.push(
      <RootItem
        indexes={[rootIndex]}
        datum={rootItem}
        data={data}
        x={/* Calculate x */}
        y={/* Calculate y */}
      />
    );

    // Iterate child nodes
    children.forEach((child, childIndex) => {
      itemElements.push(
        <ChildItem
          indexes={[rootIndex, childIndex]}
          datum={child}
          data={data}
          x={/* Calculate x */}
          y={/* Calculate y */}
        />
      );
    });
  });

  return (
    <FlexLayout
      id="infographic-container"
      flexDirection="column"
      justifyContent="center"
      alignItems="center"
    >
      {titleContent}
      <Group>
        <ItemsGroup>{itemElements}</ItemsGroup>
        <BtnsGroup>{btnElements}</BtnsGroup>
      </Group>
    </FlexLayout>
  );
};

registerStructure('[structure-name]', {
  component: [StructureName],
  composites: ['title', 'item'], // Fill in based on the components actually used.
});
```

**Relationship Structure Template:**

Relationship structure templates can access relationship data via `data.relations`.

### 5. Component Declaration (composites)

**composites Field Description:**

When calling `registerStructure`, a `composites` array must be provided to declare which core components the structure uses. This helps the system understand the structure's composition and dependencies.

**composites Value Rules:**

The values in the composites array should be lowercase strings, including the following:

1. **'title'** - Included when any of the following conditions are met:
   - The `Title` prop component is used (from `props.Title`).
   - `data.title` is directly accessed and rendered (e.g., using `<Text>{title}</Text>` or `<Text>{data.title}</Text>`).
   - The title data is rendered as a UI element in any way in the code.

2. **'item'** - Included when any of the following conditions are met:
   - The `Item` prop component is used (from `props.Item`).
   - The `Items` prop component array is used (from `props.Items`).
   - Note: Even if `Items` (plural) is used, `'item'` (singular) should be written in composites.

3. **'illus'** - Included when any of the following conditions are met:
   - The `Illus` component is used (imported from `../components`).
   - `data.illus` is directly accessed and rendered (e.g., via an image or SVG element).
   - `data.illus.xxx` is accessed and rendered as UI.

**Example**:

```tsx
// Example 1: Title and Item props used.
registerStructure('list-row', {
  component: ListRow,
  composites: ['title', 'item'],
});

// Example 2: Title rendered directly in code, Item prop used.
registerStructure('list-sector', {
  component: ListSector,
  composites: ['title', 'item'], // Although the Title prop is not used, data.title is rendered.
});

// Example 3: Items array used (hierarchical structure).
registerStructure('hierarchy-tree', {
  component: HierarchyTree,
  composites: ['title', 'item'], // Note: use 'item', not 'items'.
});

// Example 4: Title, Item, and Illus used.
registerStructure('some-structure', {
  component: SomeStructure,
  composites: ['title', 'item', 'illus'],
});
```

**IMPORTANT:**

- Values in the composites array must be lowercase.
- Even if `Items` (plural) is used, write `'item'` (singular).
- If the structure directly renders a data field (e.g., `data.title`), it should be declared in composites even if the corresponding prop component is not used.
- The composites array cannot be empty; it must contain at least `['item']`.

### 6. Key Constraints

**Strictly Follow These Rules:**

1. **ONLY use the components listed above; do not import or use unlisted components (e.g., Circle, Line).**
2. **All graphical components MUST use x/y/width/height for positioning; do not use native SVG attributes like cx/cy/r/rx/ry.**
3. **Polygon's points MUST be an object array `{x: number, y: number}[]`, not a string.**
4. **Text component's text content is passed as children; do not use the text attribute.**
5. **ALL button components MUST pass the indexes array.**
6. **Coordinate calculations MUST be based on the return value of getElementBounds.**
7. **Determine the overall layout based on Item position and size; avoid negative coordinate values.**
8. **Decoration elements (lines, arrows, etc.) should be in a separate Group and placed before ItemsGroup to ensure the decoration layer is below the data items layer.**
9. **When using the Items array, obtain components at different levels via destructuring, e.g., `const [RootItem, ChildItem] = Items`.**
10. **Certain special structures may not use buttons (e.g., network graphs, quadrants), depending on requirements.**
11. **Use Themes and Palettes:**
    - Prioritize using `getPaletteColor(options, indexes)` for data item colors.
    - Use `getColorPrimary(options)` for decoration elements.
    - Use `getThemeColors` for the full theme configuration.
12. **Group supports the transform attribute, but use it cautiously.**
13. **Native SVG elements (e.g., `<linearGradient>`, `<stop>`) can be used inside Defs to create effects like gradients.**
14. **For complex layout calculations, use d3 algorithms (e.g., `d3.hierarchy`, `d3.tree`, `d3.forceSimulation`).**
15. **Empty Data Handling: When `items.length === 0`, provide a friendly empty state (e.g., a single add button).**

### 7. Button Layout Principles

**BtnAdd (Add Button):**

- Placed between two data items, indicating a new item can be inserted here.
- The first BtnAdd is before the first data item.
- The last BtnAdd is after the last data item.
- The indexes value is the index of the insertion position (e.g., inserting before item 0, `indexes=[0]`).

**BtnRemove (Remove Button):**

- Placed near each data item, indicating the item can be removed.
- The indexes value is the index of the corresponding data item.

**Position Calculation Examples:**

- **Horizontal Layout**: BtnAdd is horizontally centered below the data item; BtnRemove is directly below.
- **Vertical Layout**: BtnAdd is horizontally centered above or below; BtnRemove is on the left or right.
- **Other Layouts**: Adjust flexibly based on visual balance and interaction convenience.

### 8. Layout Calculation Key Points

- **Element Sizing**: Use `getElementBounds()` for dimension calculations.
- **Coordinate System**: positive x is to the right, positive y is downward.
- **Item Alignment**: `positionH` and `positionV` control alignment of content within the element.
  - `positionH`: 'normal' (Default design) | 'center' (Horizontally centered) | 'flipped' (Flipped layout)
  - `positionV`: 'normal' (Default design) | 'middle' (Vertically centered) | 'flipped' (Flipped layout)
  - Example: For circular distribution, the right-side Item uses 'normal', the left side uses 'flipped'.
- **Item Size Constraints**: Some structures require limiting Item size, passed via `width` and `height` attributes.
- **Layout Mode**:
  - Simple layouts using `FlexLayout` can auto-center and arrange.
  - Complex layouts require manual calculation of precise coordinates for each element.
- **Decoration Element Layer**: Decoration elements (lines, arrows) should be in an independent Group and placed before ItemsGroup.
- **Prioritize d3 for Complex Layouts:**
  - Tree layout: `d3.tree()` or `d3.cluster()`
  - Force-directed layout: `d3.forceSimulation()`
  - Hierarchical data: `d3.hierarchy()`

### 9. Naming Conventions

> Supported Types: List, Compare, Sequence, hierarchy, relation, geo, chart.

- **Component Name**: PascalCase, e.g., `ListRow`, `CompareLeftRight`.
- **Registration Name**: kebab-case, consistent with classification prefix, e.g., `list-row`, `list-column`.
- **Props Interface**: Component Name + `Props`, e.g., `ListRowProps`.
- **Variable Naming**: Use meaningful names, e.g., `itemElements`, `btnElements`, `decorElements`.

### 10. Parameter Design Guidance

**Common Parameters and Default Values**:

- `gap`: Item spacing, default 20-40 (for list, sequence structures).
- `rowGap` / `columnGap`: Row/column gap.
- `spacing`: Overall spacing, default 20-30.
- `radius`: Circular layout radius, default 150-250.
- `outerRadius` / `innerRadius`: Outer/inner radius (ring layout).
- `angle` / `startAngle` / `endAngle`: Angle-related parameters.
- `columns` / `rows`: Number of columns/rows for grid layout, default 3-4.
- `itemsPerRow`: Items per row, default 3.
- `levelGap`: Hierarchy gap, default 60-80.
- `showAxis` / `showConnections`: Whether to show axis/connection lines, default true.

**Parameter Design Principles**:

- All parameters should have reasonable default values.
- Use optional parameter `?` marker.
- Parameter names should clearly express meaning.
- Boolean parameters use `show*` / `enable*` prefixes.

## Code Generation Requirements

1. **Completeness**:
   - Generate complete, runnable code including all required imports, type definitions, and registration statements.
   - Only import components and functions actually used.
   - **MUST include the composites array in the registerStructure call**, correctly declaring used components.

2. **Correctness**:
   - Ensure `indexes` array is correctly passed to all required components.
   - Accurate coordinate calculations to avoid overlapping or misaligned elements.
   - Handle edge cases (e.g., provide a friendly empty state for empty items).
   - Use `getElementBounds` for accurate element dimensions.
   - Text component's text is passed via children, not the text attribute.
   - **The composites array MUST accurately reflect the components actually used.** (See "Component Declaration (composites)" section.)

3. **Simplicity**:
   - Use meaningful but concise variable names.
   - Avoid redundant calculations; reuse results reasonably.
   - Extract constants and configuration items.

4. **Consistency**:
   - Follow the style and patterns of example code.
   - Button layout logic matches structure type.
   - Obtain theme colors using utility functions.

5. **Extensibility**:
   - Reserve space for custom parameters; all parameters have reasonable default values.
   - Support nested structures (access sub-items via `datum.children` when needed).
   - Props interface inherits `BaseStructureProps`.

6. **Performance Optimization**:
   - Use `forEach` to iterate data items, not `map`.
   - Collect elements into an array for unified rendering.

7. **Other Requirements**:
   - No code comments needed (unless logic is exceptionally complex).
   - Do not use React features (e.g., `key`, `useEffect`).
   - Array elements can be passed directly as children without keys.

## Generation Process

Follow these steps when a user requests structure generation:

1. **Understand Requirements**:
   - Clarify desired layout type, characteristics, and purpose.
   - Understand data organization (flat, nested, hierarchical, etc.).
   - Confirm if button interaction is needed.

2. **Determine Classification**:
   - Classify into the appropriate structure based on characteristics.
   - Choose appropriate naming (follow conventions).

3. **Design Layout**:
   - Determine whether to use `Item` or `Items`.
   - Determine arrangement and alignment of data items.
   - Calculate positional relationships of elements.
   - Design decoration elements (lines, arrows, etc.).
   - Design reasonable button positions (if needed).

4. **Write Code**:
   - Add JSX import directives.
   - Import required components and functions.
   - Define Props interface.
   - Implement component logic.
   - **Register structure (including composites array).**

5. **Verify Output**:
   - Check code completeness and correctness.
   - Confirm all imports are correct.
   - Confirm indexes are correctly passed.
   - Confirm coordinate calculations are correct.
   - **Confirm the composites array accurately reflects used components.**

## Reference Examples

### Example 1: Simple Horizontal List

**Requirement**: Data items arranged horizontally, evenly spaced.

**Implementation Key Points**:

- Use a single `Item` component.
- Each item's x coordinate = index × (itemWidth + gap).
- Use `positionH="center"` to center content.
- BtnAdd is between adjacent items; BtnRemove is below each item.

**Key Code Snippets**:

```tsx
items.forEach((item, index) => {
  const itemX = index * (itemBounds.width + gap);
  itemElements.push(
    <Item
      indexes={[index]}
      datum={item}
      data={data}
      x={itemX}
      positionH="center"
    />,
  );
});
```

### Example 2: Hierarchical Comparison Structure

**Requirement**: Left and right columns, each with a root node and multiple child nodes.

**Implementation Key Points**:

- Use Items array: `[RootItem, ChildItem]`.
- Root node at fixed position, child nodes arranged below.
- Child nodes use different `positionH` (left column 'normal', right 'flipped').

**Key Code Snippets**:

```tsx
const [RootItem, ChildItem] = Items;
items.forEach((rootItem, rootIndex) => {
  const { children = [] } = rootItem;
  itemElements.push(
    <RootItem indexes={[rootIndex]} datum={rootItem} data={data} />,
  );

  children.forEach((child, childIndex) => {
    itemElements.push(
      <ChildItem indexes={[rootIndex, childIndex]} datum={child} data={data} />,
    );
  });
});
```

### Example 3: Sequence Structure with Decorations

**Requirement**: Horizontal flow, arrows connecting data items.

**Implementation Key Points**:

- Use decoration elements (`SimpleArrow`) to connect adjacent items.
- Decoration elements in independent `Group`, placed before `ItemsGroup`.
- Draw arrows using theme color.

**Key Code Snippets**:

```tsx
const colorPrimary = getColorPrimary(options);
items.forEach((item, index) => {
  if (index < items.length - 1) {
    decorElements.push(
      <SimpleArrow
        x={itemX + itemBounds.width + (gap - arrowWidth) / 2}
        y={itemY + itemBounds.height / 2 - arrowHeight / 2}
        width={arrowWidth}
        height={arrowHeight}
        colorPrimary={colorPrimary}
      />,
    );
  }
});

return (
  <Group>
    <Group>{decorElements}</Group>
    <ItemsGroup>{itemElements}</ItemsGroup>
    <BtnsGroup>{btnElements}</BtnsGroup>
  </Group>
);
```

### Example 4: Circular Layout Using Palette

**Requirement**: Data items in circular distribution, each with a different color.

**Implementation Key Points**:

- Use trigonometric functions for circular positioning.
- Use `getPaletteColor` for each item's color.
- Pass color to `Item` via `themeColors`.

**Key Code Snippets**:

```tsx
items.forEach((item, index) => {
  const angle = (index * 2 * Math.PI) / items.length - Math.PI / 2;
  const itemX = centerX + radius * Math.cos(angle) - itemBounds.width / 2;
  const itemY = centerY + radius * Math.sin(angle) - itemBounds.height / 2;
  const color = getPaletteColor(options, [index]);

  itemElements.push(
    <Item
      indexes={[index]}
      datum={item}
      data={data}
      x={itemX}
      y={itemY}
      themeColors={getThemeColors({ colorPrimary: color }, options)}
    />,
  );
});
```

You can creatively design new layout structures based on these patterns.

## Output Format

Generated code should be a complete TypeScript file including:

- **Type Imports**: Import necessary types like `ComponentType`, `JSXElement`.
- **Component Imports**: Import atomic, encapsulated, and decoration components as needed.
- **Utility Function Imports**: Import utility functions for layout, theme, and data processing.
- **Props Interface**: Inherit `BaseStructureProps`, define custom parameters.
- **Component Implementation**: Complete component logic.
- **Structure Registration**: Register the component using `registerStructure`.

**Code Style Requirements**:

- Use 2-space indentation.
- Group import statements by type.
- Use `const` for variable declarations.
- Use concise syntax for arrow functions.
- Use appropriate empty lines to separate logic blocks.

**Example Output**:

```tsx
import type { ComponentType, JSXElement } from '../../jsx';
import { getElementBounds, Group } from '../../jsx';
import { BtnAdd, BtnRemove, BtnsGroup, ItemsGroup } from '../components';
import { FlexLayout } from '../layouts';
import { registerStructure } from './registry';
import type { BaseStructureProps } from './types';

export interface ExampleProps extends BaseStructureProps {
  gap?: number;
}

export const Example: ComponentType<ExampleProps> = (props) => {
  // Component Implementation
};

registerStructure('example', {
  component: Example,
  composites: ['title', 'item'], // Fill in based on the components actually used.
});
```

---
