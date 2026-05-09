# Charts in Markdown

This document explains how to embed charts in documentation using Chartist.js chart components.

## Chart Types

We support the following chart types:

1. **Line Chart** - Time series, trends, comparisons
2. **Bar Chart** - Categorical comparisons, grouped data
3. **Pie Chart** - Composition, part-to-whole relationships
4. **Sparkline** - Mini trends in small spaces

## Syntax

Use ```chart code blocks with JSON configuration:

```markdown
\`\`\`chart
{
  "type": "line|bar|pie|sparkline",
  "data": [...],
  "labels": [...],
  "width": 600,
  "height": 300,
  "color": "#hexcolor",
  ...additional_options
}
\`\`\`
```

## Line Chart

Shows trends over time or ordered categories.

```markdown
\`\`\`chart
{
  "type": "line",
  "data": [10, 20, 15, 30, 25],
  "labels": ["Jan", "Feb", "Mar", "Apr", "May"],
  "width": 600,
  "height": 300,
  "color": "#E99339",
  "showLine": true,
  "showArea": true,
  "showPoint": true,
  "smooth": true
}
\`\`\`
```

### Line Chart Options

| Option | Type | Default | Description |
|--------|-------|----------|-------------|
| `type` | string | required | Must be `"line"` |
| `data` | `number[] \| number[][]` | required | Single or multiple series |
| `labels` | `string[]` | `["0", "1", ...]` | X-axis labels |
| `width` | number | `300` | Chart width (px) |
| `height` | number | `200` | Chart height (px) |
| `color` | `string \| string[]` | `["#E99339"]` | Single or multiple colors |
| `showLine` | boolean | `true` | Show line path |
| `showArea` | boolean | `false` | Fill area under line |
| `showPoint` | boolean | `true` | Show data points |
| `smooth` | boolean | `false` | Use bezier curves |
| `stacked` | boolean | `false` | Stack multiple series |

## Area Chart

Same as line chart but with `showArea: true`:

```markdown
\`\`\`chart
{
  "type": "line",
  "data": [10, 20, 15, 30, 25],
  "showArea": true,
  "showPoint": false,
  "color": "#3d7eff",
  "smooth": true
}
\`\`\`
```

## Scatter Chart

Line chart with only points, no connecting line:

```markdown
\`\`\`chart
{
  "type": "line",
  "data": [10, 20, 15, 30, 25],
  "showLine": false,
  "showPoint": true,
  "color": "#10B981"
}
\`\`\`
```

## Bar Chart

Shows categorical comparisons.

```markdown
\`\`\`chart
{
  "type": "bar",
  "data": [100, 200, 150, 250, 180],
  "labels": ["A", "B", "C", "D", "E"],
  "width": 600,
  "height": 300,
  "color": "#E99339"
}
\`\`\`
```

### Bar Chart Options

| Option | Type | Default | Description |
|--------|-------|----------|-------------|
| `type` | string | required | Must be `"bar"` |
| `data` | `number[] \| number[][]` | required | Single or multiple series |
| `labels` | `string[]` | `["0", "1", ...]` | X-axis labels |
| `width` | number | `300` | Chart width (px) |
| `height` | number | `200` | Chart height (px) |
| `color` | `string \| string[]` | `["#E99339"]` | Single or multiple colors |
| `horizontal` | boolean | `false` | Display bars horizontally |
| `stacked` | boolean | `false` | Stack multiple series |

## Pie Chart

Shows composition/part-to-whole relationships.

```markdown
\`\`\`chart
{
  "type": "pie",
  "data": [30, 50, 20],
  "labels": ["BTC", "ETH", "SOL"],
  "width": 300,
  "height": 300,
  "color": ["#E99339", "#3d7eff", "#10B981"],
  "donut": true,
  "donutWidth": 20
}
\`\`\`
```

### Pie Chart Options

| Option | Type | Default | Description |
|--------|-------|----------|-------------|
| `type` | string | required | Must be `"pie"` |
| `data` | `number[]` | required | Series data values |
| `labels` | `string[]` | `["0", "1", ...]` | Slice labels |
| `width` | number | `300` | Chart width (px) |
| `height` | number | `300` | Chart height (px) |
| `color` | `string \| string[]` | `["#E99339"]` | Slice colors |
| `donut` | boolean | `false` | Display as donut |
| `donutWidth` | number | `20` | Donut ring thickness |

## Sparkline

Mini trend indicator, no axes.

```markdown
\`\`\`chart
{
  "type": "sparkline",
  "data": [10, 20, 15, 30, 25, 35, 28, 40],
  "width": 120,
  "height": 40,
  "color": "#10B981",
  "showArea": true,
  "smooth": true
}
\`\`\`
```

### Sparkline Options

Same as line chart, but axes are always hidden.

## Multi-Series Charts

Compare multiple series by passing 2D array:

```markdown
\`\`\`chart
{
  "type": "line",
  "data": [
    [10, 20, 15, 30, 25],
    [5, 15, 25, 20, 35]
  ],
  "labels": ["Jan", "Feb", "Mar", "Apr", "May"],
  "color": ["#E99339", "#3d7eff"],
  "showArea": true
}
\`\`\`
```

## Stacked Charts

Stack multiple series:

```markdown
\`\`\`chart
{
  "type": "bar",
  "data": [
    [10, 20, 30],
    [20, 10, 15]
  ],
  "stacked": true,
  "color": ["#E99339", "#3d7eff"]
}
\`\`\`
```

## Colors

Use any valid CSS color format:

- Hex: `"#E99339"`
- RGB: `"rgb(233, 147, 57)"`
- Named: `"orange"`

For multi-series, pass array:

```json
["#E99339", "#3d7eff", "#10B981"]
```

## Best Practices

1. **Keep charts simple** - Avoid cluttering with too much data
2. **Use consistent colors** - Brand colors: `#E99339` (primary), `#3d7eff` (secondary)
3. **Responsive widths** - Use percentage or reasonable fixed widths (400-800px)
4. **Label readability** - Keep labels short and meaningful
5. **Chart type selection**:
   - Time series → Line chart
   - Categories → Bar chart
   - Composition → Pie chart
   - Small trend → Sparkline

## Example: Cumulative Emission Chart

```markdown
\`\`\`chart
{
  "type": "line",
  "data": [0, 32.5, 49.8, 57.9, 61.9, 63.9],
  "labels": ["Year 0", "Year 2", "Year 4", "Year 6", "Year 8", "Year 10+"],
  "width": 700,
  "height": 350,
  "color": "#E99339",
  "showArea": true,
  "showPoint": true
}
\`\`\`
```

## Example: Supply Allocation Pie

```markdown
\`\`\`chart
{
  "type": "pie",
  "data": [65, 20, 12, 3],
  "labels": ["Emissions", "Treasury", "Team", "LBP"],
  "width": 400,
  "height": 400,
  "color": ["#E99339", "#3d7eff", "#10B981", "#ffd61e"],
  "donut": true
}
\`\`\`
```

## Technical Notes

- Charts are rendered at runtime using Preact components
- Configuration is stored in `data-chart-config` attribute
- Invalid JSON falls back to showing error message
- Charts are responsive within their containers
- Light/dark theme support via CSS custom properties

## Troubleshooting

### Chart doesn't render

1. Check JSON syntax - validate with a JSON linter
2. Ensure required fields: `type`, `data`
3. Verify data is array of numbers or 2D array

### Colors not showing

1. Ensure color values are valid CSS colors
2. For multi-series, use array of colors

### Wrong chart type

1. Verify `type` matches expected: `"line"`, `"bar"`, `"pie"`, `"sparkline"`
