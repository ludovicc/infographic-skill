# Infographic Data Item Component Generation Specification

This document is used to guide the generation of Item component code that complies with the framework specifications.

## Table of Contents
- Core Concepts of Data Items
- Data Item Design Philosophy
- Technical Specifications
- Code Generation Requirements
- Generation Process
- Output Format
- FAQ and Best Practices

## Core Concepts of Data Items

A data item (Item) is the basic information unit in an infographic, responsible for displaying a single data element. Data items are organized and laid out through structures (Structure) to form a complete infographic.

Each data item component receives:

- **datum**: The data object for the current data item
- **data**: The complete data set (containing all items)
- **indexes**: The position index of the current data item in the structure
- **themeColors**: Theme color configuration
- **positionH**: Horizontal alignment (supports 'normal', 'center', 'flipped')
- **positionV**: Vertical alignment (supports 'normal', 'middle', 'flipped')

## Data Item Design Philosophy

Data items do not have a fixed classification system; they are created flexibly based on the design requirements of different infographics. They can be simple text displays, complex chart elements, geometric shapes with special forms, or any other visual representation. When designing, consider:

- Information display requirements (text, values, icons, status, etc.)
- Visual hierarchy and aesthetics
- Compatibility with structural layout
- Reasonable use of theme colors

### Design Requirements

- **Completeness**: Data items should support four basic elements: ItemIcon, ItemLabel, ItemDesc, ItemValue. All elements are optional.
- **Adaptive Layout**: When some elements are missing, others should automatically adjust their positions to make full use of the space.
- **Numerical Handling**: `datum.value` may be undefined; it needs to be handled correctly and conditionally rendered.
- **Gradient Usage**: Not all designs require gradients; the decision should be based on visual effects, as solid colors are equally effective.

## Technical Specifications

### 1. Type Definitions

```typescript
export interface BaseItemProps {
  x?: number;
  y?: number;
  id?: string;
  indexes: number[];
  data: Data;
  datum: Data['items'][number];
  themeColors: ThemeColors;
  positionH?: 'normal' | 'center' | 'flipped';
  positionV?: 'normal' | 'middle' | 'flipped';
  valueFormatter?: (value: number) => string | number;
  [key: string]: any;
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

export interface ThemeColors {
  /** Original primary color */
  colorPrimary: string;
  /** Light background for primary color */
  colorPrimaryBg: string;
  /** Text color on primary color background */
  colorPrimaryText: string;
  /** Deepest text color */
  colorText: string;
  /** Secondary text color */
  colorTextSecondary: string;
  /** Pure white */
  colorWhite: string;
  /** Background color */
  colorBg: string;
  /** Card background color */
  colorBgElevated: string;
}
```

### 2. Available Components List

**Atomic Components (Imported from ../../jsx)**

Uniformly use `x`, `y`, `width`, `height` attributes for positioning and sizing:

- **Defs**: SVG definition container, used to define reusable SVG elements such as gradients and patterns.

- **Rect**: Rectangle

> Other attributes are consistent with React SVG rect.

- **Ellipse**: Ellipse/Circle

> Other attributes are consistent with React SVG ellipse, but use `x/y/width/height` for coordinates instead of `cx/cy/rx/ry`.

- **IMPORTANT**: `x`, `y` are the top-left position of the ellipse, not the center coordinates.

- **Path**: Path shape

> Other attributes are consistent with React SVG path.

- **Polygon**: Polygon

  ```typescript
  import { Point } from '../../jsx';
  <Polygon points={[{x: 0, y: 0}, {x: 100, y: 0}, {x: 50, y: 100}]} fill={color} />
  ```

> Other attributes are consistent with React SVG polygon.

- **Group**: Grouping container

  ```typescript
  <Group x={0} y={0} width={100} height={100}>
    {children}
  </Group>
  ```

  > The width and height of a Group do not impose any constraints; they are only used to obtain the bounding box. If not set, the bounding box will be calculated based on child nodes.
  > Other attributes are consistent with React SVG group.

- **ShapesGroup**

Attributes and usage are identical to Group, but internal shapes can be stylized.

- **Text**: Text

  ```typescript
  <Text x={0} y={0} width={100} height={20} fontSize={14} fill={color}>
    Content
  </Text>
  ```

  Extended Attributes:
  - **alignHorizontal**: "left" | "center" | "right", horizontal alignment position
  - **alignVertical**: "top" | "middle" | "bottom", vertical alignment position
  - **lineHeight**: Line height, default 1.2
  - **wordWrap**: Whether to wrap text, default false
  - **backgroundColor**: Background color

> Other attributes are consistent with React SVG text.

**Encapsulated Components (Imported from ../components)**

- **ItemIcon**: Data item icon (square)

  ```typescript
  <ItemIcon
    indexes={indexes}
    x={0}
    y={0}
    size={30}
    fill={themeColors.colorPrimary}
  />
  ```

- **ItemIconCircle**: Data item icon (circular background container)

  ```typescript
  <ItemIconCircle
    indexes={indexes}
    x={0}
    y={0}
    size={50}
    fill={themeColors.colorPrimary}      // Circular background color
    colorBg={themeColors.colorWhite}     // Internal icon background color
  />
  ```

- **ItemLabel**: Data item label (with default styling)

  ```typescript
  <ItemLabel
    indexes={indexes}
    x={0}
    y={0}
    width={100}
    // Unless special styling is required, it is not recommended to set the following attributes
    // fontSize={14}
    // alignHorizontal="center"
    // alignVertical="middle"
    // fill={themeColors.colorText}
  >
    {datum.label}
  </ItemLabel>
  ```

- **ItemDesc**: Data item description (with default styling)

  ```typescript
  <ItemDesc
    indexes={indexes}
    x={0}
    y={0}
    width={200}
    // Unless special styling is required, it is not recommended to set the following attributes
    // fontSize={12}
    // lineHeight={1.4}
    // lineNumber={2}
    // wordWrap={true}
    // fill={themeColors.colorTextSecondary}
  >
    {datum.desc}
  </ItemDesc>
  ```

- **ItemValue**: Data item value (with default styling)

  ```typescript
  <ItemValue
    indexes={indexes}
    x={0}
    y={0}
    value={datum.value}
    formatter={extractedProps.valueFormatter}  // Optional formatting function
    // Unless special styling is required, it is not recommended to set the following attributes
    // fontSize={16}
    // fontWeight="bold"
    // fill={themeColors.colorPrimary}
  />
  ```

- **Illus**: Illustration component

  ```typescript
  <Illus
    x={0}
    y={0}
    width={100}
    height={100}
  />
  ```

- **Gap**: Layout spacing placeholder

  ```typescript
  <Gap width={10} height={10} />
  ```

  - **IMPORTANT**: Can only be used directly as `<Gap />`, not via `const gap = <Gap />`.

**Layout Components (Imported from ../layouts)**

- **FlexLayout**: Flex layout

  ```typescript
  <FlexLayout
    flexDirection="row" | "column"
    gap={8}
    alignItems="flex-start" | "center" | "flex-end"
  >
    {children}
  </FlexLayout>
  ```

- **AlignLayout**: Alignment layout

> For example, child elements can be aligned horizontally and vertically (elements may overlap).
> Alignment can also be performed separately for horizontal or vertical, with the position in the other direction remaining unchanged.

```typescript
<AlignLayout
  horizontal="left" | "center" | "right"
  vertical="top" | "middle" | "bottom"
  width={100}   // Optional, alignment container size
  height={100}  // Optional, alignment container size
>
  {children}
</AlignLayout>
```

### 3. Utility Functions

- **getElementBounds**: Get element bounds

  ```typescript
  const bounds = getElementBounds(<ItemLabel indexes={indexes} />);
  // Returns: { x: number, y: number, width: number, height: number }
  ```

- **getItemProps**: Extract and handle props, with the second parameter being a list of custom attribute names.

  ```typescript
  // Extract specified custom attributes from props to avoid passing them to restProps
  const [extractedProps, restProps] = getItemProps(props, [
    'width',
    'height',
    'iconSize',
  ]);
  // extractedProps: An object containing all BaseItemProps + specified custom attributes
  // restProps: Remaining props, usually passed to the outermost Group (to avoid DOM warnings)
  ```

- **getItemKeyFromIndexes**: Generate key from index array
  ```typescript
  import { getItemKeyFromIndexes } from '../../utils';
  const key = getItemKeyFromIndexes([0, 1]); // "0-1"
  ```

### 4. Third-Party Library Support

The following libraries can be imported to enhance functionality:

- **d3**:

  ```typescript
  import { xxx } from 'd3';
  ```

- **lodash-es**: Utility functions (recommended to import as needed)

  ```typescript
  import { xxx } from 'lodash-es';
  ```

- **tinycolor2**: Color handling

  ```typescript
  import tinycolor from 'tinycolor2';

  // Instance methods - Chained calls
  tinycolor(color).darken(20).toHexString();
  tinycolor(color).lighten(10).toHexString();

  // Static methods - Mix colors
  tinycolor.mix(themeColors.colorPrimary, '#fff', 40).toHexString();

  // Clone method - Avoid modifying the original object
  const base = tinycolor(baseColor);
  const gradStart = base.clone().darken(4).toHexString();
  const gradEnd = base.clone().lighten(12).toHexString();
  ```

- **round-polygon**: Rounded polygon processing
  ```typescript
  import roundPolygon, { getSegments } from 'round-polygon';
  const rounded = roundPolygon(points, radius);
  const segments = getSegments(rounded, 'AMOUNT', 10);
  ```

### 5. Import Template

```typescript
import { ComponentType, Group } from '../../jsx';

// Selectively import atomic components and types as needed
import {
  getElementBounds,
  Defs,
  Ellipse,
  Path,
  type Point, // Point type required for Polygon
  Polygon,
  Rect,
  Text,
} from '../../jsx';

// Selectively import encapsulated components as needed
import {
  Gap,
  Illus,
  ItemDesc,
  ItemIcon,
  ItemIconCircle,
  ItemLabel,
  ItemValue,
} from '../components';

// Selectively import layout components as needed
import { AlignLayout, FlexLayout } from '../layouts';

import { registerItem } from './registry';
import type { BaseItemProps } from './types';
import { getItemProps } from './utils';

// Import third-party libraries as needed
// import { xxx } from 'd3';
// import tinycolor from 'tinycolor2';
// import { xxxx } from 'lodash-es';
// import roundPolygon, { xxx } from 'round-polygon';
```

### 6. composites Field Rules

When calling `registerItem`, a `composites` field must be passed to indicate which encapsulated components are used by the current component. The value of `composites` is determined based on the components used in the code implementation:

- **ItemLabel** → `"label"`
- **ItemDesc** → `"desc"`
- **ItemValue** → `"value"`
- **ItemIcon** or **ItemIconCircle** → `"icon"`
- **Illus** → `"illus"`

**Example**:

```typescript
// If the component uses ItemLabel and ItemDesc
registerItem('simple-text', {
  component: SimpleText,
  composites: ['label', 'desc'],
});

// If the component uses ItemIcon, ItemLabel, ItemValue, and ItemDesc
registerItem('full-card', {
  component: FullCard,
  composites: ['icon', 'label', 'value', 'desc'],
});

// If the component uses Illus, ItemLabel, and ItemDesc
registerItem('illus-item', {
  component: IllusItem,
  composites: ['illus', 'label', 'desc'],
});
```

### 7. Component Structure Template

```typescript
export interface [ItemName]Props extends BaseItemProps {
  // Custom parameters in addition to BaseItemProps (gap, etc., customized according to design needs)
  width?: number;
  height?: number;
  iconSize?: number;
  // Other custom parameters
}

export const [ItemName]: ComponentType<[ItemName]Props> = (props) => {
  const [
    {
      datum,
      data,
      indexes,
      width = 300,
      height = 60,
      iconSize = 30,
      positionH = 'normal',
      positionV = 'normal',
      themeColors,
      valueFormatter = (v: any) => `${v}%`,  // Can set a default formatting function
      // Other custom parameters
    },
    restProps,
  ] = getItemProps(props, ['width', 'height', 'iconSize' /* other custom parameters */]);

  // 1. Data Handling
  const value = datum.value; // Keep original value for conditional logic
  const displayValue = value ?? 0; // Value for display

  // 2. Size and Position Calculation
  // Use getElementBounds to get child element sizes
  // Adjust layout according to positionH/positionV

  // 3. Gradient Definition (if needed)
  const gradientId = `${themeColors.colorPrimary}-component-name`; // Generated based on color for easy reuse

  // 4. Component Structure
  return (
    <Group {...restProps}>
      {/* Defs definition (if gradient is needed) */}

      {/* Main shape and content */}
      {datum.icon && <ItemIcon indexes={indexes} {...iconProps} />}

      {datum.label !== undefined && (
        <ItemLabel
          indexes={indexes}
          x={/** Calculate X coordinate */}
          y={/** Calculate Y coordinate */}
          {...labelProps}>
          {datum.label}
        </ItemLabel>
      )}

      {/* Value - Conditional rendering */}
      {value !== undefined && (
        <ItemValue
          indexes={indexes}
          x={/** Calculate X coordinate */}
          y={/** Calculate Y coordinate */}
          value={displayValue}
          formatter={valueFormatter}
          {...valueProps}
        />
      )}

      {/* Description - Dynamic layout */}
      {datum.desc !== undefined && (
        <ItemDesc
          indexes={indexes}
          x={/** Calculate X coordinate */}
          y={/** Calculate Y coordinate */}
          {...descProps}
        >
          {datum.desc}
        </ItemDesc>
      )}
    </Group>
  );
};

registerItem('[item-name]', {
  component: [ItemName],
  composites: ['label', 'desc', 'icon', 'value', 'illus'] // Determined based on the components actually used by the component
});
```

### 8. indexes Index System

**indexes** is the position identifier for a data item in an infographic, using an array to represent hierarchical relationships:

- **One-dimensional structure** (lists, rows, etc.): `[0]`, `[1]`, `[2]`, ...
- **Nested structure** (trees, hierarchies, etc.):
  - Root node: `[0]`
  - Child nodes of the first root node: `[0, 0]`, `[0, 1]`, `[0, 2]`, ...
  - Child nodes of node `[0, 1]`: `[0, 1, 0]`, `[0, 1, 1]`, ...

This index system ensures that each data item has a unique identifier.

**Common operations on indexes**:

```typescript
// Generate sequence number
const indexNumber = indexes[0] + 1;
const indexStr = String(indexes[0] + 1).padStart(2, '0'); // "01", "02", ...

// Check for even/odd (for alternating styles)
const isEven = indexes[0] % 2 === 0;
```

### 9. Key Design Principles

#### Handling positionH/positionV

> It's not always necessary to handle positionH/V, but if the design has alignment requirements, adaptation is needed.

```typescript
// Example of handling positionH
const iconX =
  positionH === 'flipped'
    ? width - iconSize // Right-aligned
    : positionH === 'center'
      ? (width - iconSize) / 2 // Centered
      : 0; // Default left-aligned

// Example of handling positionV
const iconY =
  positionV === 'middle'
    ? (height - iconSize) / 2
    : positionV === 'flipped'
      ? height - iconSize
      : 0;

// Text alignment mode
const textAlign =
  positionH === 'flipped'
    ? 'right'
    : positionH === 'center'
      ? 'center'
      : 'left';
```

#### Using Theme Colors

```typescript
// Primary color tone
fill={themeColors.colorPrimary}

// Background color
fill={themeColors.colorPrimaryBg}

// Main text
fill={themeColors.colorText}

// Secondary text
fill={themeColors.colorTextSecondary}

// White (often used for icons/text on dark backgrounds)
fill={themeColors.colorWhite}
```

#### Gradient Definition

**Linear Gradient**:

```typescript
const gradientId = `${themeColors.colorPrimary}-component-name`;
<Defs>
  <linearGradient id={gradientId} x1="0%" y1="0%" x2="100%" y2="0%">
    <stop offset="0%" stopColor={themeColors.colorPrimary} />
    <stop
      offset="100%"
      stopColor={tinycolor.mix(themeColors.colorPrimary, '#fff', 40).toHexString()}
    />
  </linearGradient>
</Defs>;

// Using the gradient
<Rect fill={`url(#${gradientId})`} {...props} />;
```

**Radial Gradient**:

```typescript
const radialId = `${themeColors.colorPrimary}-radial`;
<Defs>
  <radialGradient id={radialId} cx="50%" cy="50%" r="50%">
    <stop offset="0%" stopColor={themeColors.colorPrimary} />
    <stop
      offset="100%"
      stopColor={tinycolor(themeColors.colorPrimary).darken(20).toHexString()}
    />
  </radialGradient>
</Defs>;
```

**Multiple Gradient Definitions**:

```typescript
// Define multiple gradients for different purposes
const progressGradientId = `${themeColors.colorPrimary}-progress`;
const backgroundGradientId = `${themeColors.colorPrimaryBg}-progress-bg`;
const positiveGradient = `gradient-${themeColors.colorPrimary}-positive`;
const negativeGradient = `gradient-${themeColors.colorPrimary}-negative`;

<Defs>
  <linearGradient id={progressGradientId} x1="0%" y1="0%" x2="100%" y2="0%">
    <stop offset="0%" stopColor={themeColors.colorPrimary} />
    <stop
      offset="100%"
      stopColor={tinycolor.mix(themeColors.colorPrimary, '#fff', 20).toHexString()}
    />
  </linearGradient>
  <linearGradient id={backgroundGradientId} x1="0%" y1="0%" x2="100%" y2="0%">
    <stop offset="0%" stopColor={themeColors.colorPrimaryBg} />
    <stop offset="100%" stopColor={themeColors.colorBg} />
  </linearGradient>
</Defs>;
```

**SVG Pattern Fill**:

```typescript
// Generate unique ID based on index (for scenarios where uniqueness is required)
const uniqueId = `letter-card-${indexes.join('-')}`;
const patternId = `${uniqueId}-pattern`;

<Defs>
  <pattern
    id={patternId}
    patternUnits="userSpaceOnUse"
    width={10}
    height={10}
    patternTransform={`rotate(45)`}
  >
    {/* NOTE: Inside pattern, use lowercase native SVG elements */}
    <rect x="0" y="0" width={4} height={10} fill="rgba(0, 0, 0, 0.03)" />
  </pattern>
</Defs>;

// Using the pattern
<Rect fill={`url(#${patternId})`} {...props} />;
```

#### Stylized Rendering Support

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

#### Responsive Sizing

```tsx
// Dynamic adjustment based on content
const labelBounds = getElementBounds(
  <ItemLabel indexes={indexes}>{datum.label}</ItemLabel>,
);
const totalHeight = iconSize + gap + labelBounds.height;

// Calculate proportions based on data set
const values = data.items.map((item) => item.value ?? 0);
const maxValue = Math.max(...values);
const barHeight = (value / maxValue) * availableHeight;
```

#### Complex SVG Path Drawing

```typescript
// 1/4 arc path
const quarterCirclePath = isFlipped
  ? `M ${x} ${y} L ${x} ${y + r} A ${r} ${r} 0 0 0 ${x + r} ${y} Z`
  : `M ${x} ${y} L ${x} ${y + r} A ${r} ${r} 0 0 1 ${x - r} ${y} Z`;

<Path d={quarterCirclePath} fill={themeColors.colorPrimary} />Simplified;

// Arrow polygon
<Polygon
  points={[
    { x: 0, y: 0 },
    { x: width - 10, y: 0 },
    { x: width, y: height / 2 }, // Arrow tip
    { x: width - 10, y: height },
    { x: 0, y: height },
    { x: 10, y: height / 2 },
  ]}
  fill={themeColors.colorPrimary}
/>;
```

**SVG Path Command Reference**:

- `M x y` - Move to
- `L x y` - Line to
- `A rx ry x-axis-rotation large-arc-flag sweep-flag x y` - Arc
- `Z` - Close path

### 10. Constraint Rules

**Strictly Follow:**

1. **Only use listed components and attributes.**
2. **All graphical components use x/y/width/height for positioning.**
3. **Must pass indexes to all encapsulated components.** (ItemIcon, ItemLabel, ItemDesc, ItemValue, etc.)
4. **Use getItemProps to handle props.**
5. **Correct use of tinycolor:**
   - Instance methods: `tinycolor(color).darken(20).toHexString()`
   - Static methods: `tinycolor.mix(color1, color2, amount).toHexString()`
6. **Support positionH/V alignment** (according to design needs).
7. **Avoid cases where element coordinates are negative.**
8. **Conditionally render optional elements** (icon, label, desc, value).
9. **Must provide composites field when registering component**, determined by encapsulated components used.

### 11. Naming Conventions

- Component Name: PascalCase, such as `DoneList`, `ChartColumn`
- Registration Name: kebab-case, such as `done-list`, `chart-column`
- Props interface: Component Name + `Props`
- Constants: UPPER_SNAKE_CASE, such as `CIRCLE_MASS`, `DOT_RADIUS`

## Code Generation Requirements

1. **Completeness**: Include all imports, type definitions, component implementations, and registration.
2. **Correctness**:
   - Only use allowed components and attributes.
   - indexes are correctly passed to all encapsulated components.
   - Accurate coordinate calculation.
   - Ellipse's x/y are top-left coordinates.
3. **Styling Principles**:
   - Prioritize default styling for ItemLabel/ItemDesc/ItemValue.
   - Only override style attributes when special effects are truly needed.
   - Reasonable use of theme colors.
4. **Flexibility**:
   - Reasonable default values for parameters.
   - Handle edge cases (empty data, missing fields, etc.).
   - Conditional rendering for optional elements.
   - Support positionH/V alignment.
   - Responsive sizing design.

## Generation Process

1. **Understand Requirements**: Clarify content and visual effects for the data item.
2. **Design Layout**: Determine element arrangement and sizing relationships.
3. **Write Code**: Generate complete code based on the template.
4. **Verify Output**: Check code completeness and compliance.

## Output Format

Generate a complete TypeScript file:

1. JSX import comments
2. Import statements
3. Props interface
4. Component implementation
5. Registration statement

## FAQ and Best Practices

### Numerical Handling Issues

Incorrect:

```typescript
const value = datum.value ?? 0; // value is never undefined
{
  value !== undefined && <ItemValue value={value} />; // condition is always true
}
```

Correct:

```typescript
const value = datum.value; // Keep original value
const displayValue = value ?? 0; // Used for display
{
  value !== undefined && <ItemValue value={displayValue} />; // Conditional rendering works correctly
}
```

### Gradient ID Generation

```typescript
// Recommended: Based on color and purpose (reusable)
const gradientId = `${themeColors.colorPrimary}-progress`;

// Or based on index (for scenarios where uniqueness is required)
const uniqueId = `letter-card-${indexes.join('-')}`;
const gradientId = `${uniqueId}-gradient`;
```

### tinycolor Usage

Incorrect:

```typescript
tinycolor.darken(color, 20); // Static method does not exist
tinycolor.lighten(color, 10); // Static method does not exist
```

Correct:

```typescript
// Instance methods - Chained calls
tinycolor(color).darken(20).toHexString();
tinycolor(color).lighten(10).toHexString();

// Static methods - Mix colors
tinycolor.mix(themeColors.colorPrimary, '#fff', 40).toHexString();

// Clone method - Avoid modifying the original object
const base = tinycolor(baseColor);
const gradStart = base.clone().darken(4).toHexString();
const gradEnd = base.clone().lighten(12).toHexString();
```

### Dynamic Layout Examples

```typescript
// Description position dynamically adjusted based on presence of value
const descY = value !== undefined ? labelY + labelHeight + valueHeight + gap : labelY + labelHeight + smallGap;

<ItemDesc indexes={indexes} y={descY}>
  {datum.desc}
</ItemDesc>;

// Content width adjusted based on presence of icon
const textWidth = showIcon && datum.icon ? width - iconSize - gap : width;

<ItemLabel indexes={indexes} width={textWidth}>
  {datum.label}
</ItemLabel>;
```

### Common Utility Functions and Techniques

```typescript
// String Formatting
String(indexes[0] + 1).padStart(2, '0'); // "01", "02", ...
datum.label?.[0].toUpperCase(); // Optional chaining + uppercase first letter

// Math Constants
Math.SQRT2; // √2 ≈ 1.414
Math.PI; // π ≈ 3.14159

// Calculate circle centroid (for arc shapes)
const CIRCLE_MASS = (4 * radius) / (3 * Math.PI);

// Circular Progress Calculation
const radius = (size - strokeWidth) / 2;
const circumference = 2 * Math.PI * radius;
const strokeDashoffset = circumference * (1 - percentage);
```
