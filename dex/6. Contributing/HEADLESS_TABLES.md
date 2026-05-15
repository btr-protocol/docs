---
title: "Headless Tables - Quick Reference"
description: "## Quick Syntax"
audience: tech
type: reference
status: planned
phase: 2
order: 99
lang: en
publish: true
---
# Headless Tables - Quick Reference

**Markdown tables without visible headers** for compact, context-aware data presentation.

---

## Quick Syntax

```markdown
| | | | |              # Empty header cells with SPACES
|---|---|---|---|      # Separator row
| Data 1 | Data 2 | Data 3 | Data 4 |
```

**CRITICAL**: Empty cells MUST contain spaces: `| | | |` not `|||||`

---

## Correct vs Incorrect

| Syntax | Works? | Example |
|--------|--------|---------|
| `| \| \| \| \|` | ✅ Yes | 4 columns with spaces |
| `\|\|\|\|\|` | ❌ No | No spaces, renders as paragraph |
| `\| \| \|` | ✅ Yes | 3 columns with spaces |
| `\|\|` | ❌ No | No spaces |

---

## Common Use Cases

### 1. Parameter Lists

```markdown
| | | | |
|---|---|---|---|
| **Operation** | **Timelock** | **Grace Period** | **Bypass** |
| `setReservePrice()` | 7 days | 0 | No |
| `setFee()` | 7 days | 0 | No |
```

### 2. Permission Mappings

```markdown
| | | |
|---|---|---|
| `admin` | `executeTimelocked()` | ✅ |
| `operator` | `setReservePrice()` | ✅ |
| `anyone` | `swap()` | ✅ |
```

### 3. Configuration Tables

```markdown
| | | |
|---|---|---|
| `slippage_tolerance` | `0.5%` | Maximum acceptable slippage |
| `min_liquidity` | `10,000` | Minimum LP token amount |
| `max_fee` | `1%` | Fee cap for swaps |
```

---

## Common Pitfalls & Fixes

| Problem | Symptoms | Fix |
|---------|----------|-----|
| No spaces in empty cells | Renders as paragraph, not table | Add spaces: `\| \| \|` |
| Wrong column count | Misaligned data or broken layout | Count pipes, match header and data cells |
| Missing separator row | Header shows as table body | Add `\|---\|---\|` row after header |
| Extra trailing pipes | Extra empty columns | Remove trailing `\|` from lines |
| Mixed empty/filled headers | Some headers visible | Use all empty or all filled headers |

---

## When to Use

| Use Headless Tables When... | Use Normal Headers When... |
|---------------------------|--------------------------|
| Column meaning is obvious from context | Column meaning needs clarification |
| 3-4 columns maximum | 5+ columns |
| Data is self-describing | Complex or unfamiliar data |
| Inline labels within data cells | Requires visual headers |
| Compact display needed | Clarity is more important |
| Section text provides context | Multiple sections with different data |

---

## Troubleshooting

| Symptom | Cause | Solution |
|---------|-------|----------|
| Table renders as paragraph | Empty header cells have no spaces | Add spaces: `\| \| \|` |
| Headers still visible | CSS not loaded or `:has()` unsupported | Check browser compatibility |
| Table layout broken | Column count mismatch | Count pipes per row |
| Sorting fails | Empty headers break detection | Use normal headers for sortable tables |

---

## Browser Compatibility

The `:has()` pseudo-class is supported in:
- ✅ Chrome 105+
- ✅ Firefox 121+
- ✅ Safari 15.4+
- ✅ Edge 105+

For older browsers, tables will display with visible (empty) headers.

---

## Technical Details

**How it works:**
1. Markdown parser (marked.js) recognizes table with empty header cells
2. HTML generated with `<thead>` containing empty `<th>` elements
3. CSS `:has()` pseudo-class detects empty headers
4. Entire `<thead>` is hidden via `display: none !important`

**CSS** (from `/front/src/styles/markdown.css`):
```css
.markdown-content table>thead:has(th:empty),
.markdown-content table>thead:has(th.sortable:empty) {
  display: none !important;
}
```

---

## Full Documentation

See [DOCUMENTATION.md](./DOCUMENTATION.md#5-headless-tables) for complete details, examples, and best practices.

---

*Last updated: 2025-01*
