# Front-End Developer Best Practices

**Tech Stack**: Preact (native) + Tailwind CSS + TradingView Lightweight Charts + Custom Router + Preact Signals + Atomic Components

---

## Quick Reference

| Area | Standard |
|------|----------|
| **Package Manager** | `bun` exclusively (never npm/yarn) |
| **Component Style** | Functional with TypeScript, native Preact |
| **State Management** | Preact Signals (class-based stores) |
| **Styling** | Tailwind CSS + CVA variants |
| **Routing** | Custom `lib/router.tsx` |
| **Charts** | TradingView Lightweight Charts |
| **Testing** | Bun test |
| **React Compatibility** | **NEVER use React or preact-compat** |

---

## 1. Native Preact Only

### Forbidden Dependencies

```json
// ❌ NEVER install or import these
{
  "dependencies": {
    "react": "FORBIDDEN",
    "react-dom": "FORBIDDEN",
    "@preact/compat": "FORBIDDEN"
  }
}
```

### Correct Imports

```typescript
// ✅ CORRECT - Native Preact imports
import { h, createContext, Context } from 'preact';
import { useState, useEffect, useRef, useMemo, useCallback } from 'preact/hooks';
import { signal, computed, effect, batch } from '@preact/signals';

// ❌ WRONG - React compatibility
import { useState } from 'react';  // NO
import { forwardRef } from 'preact/compat';  // NO

// ✅ CORRECT - Use native Preact patterns
// For ref forwarding, use ref prop directly or custom patterns
function MyComponent({ innerRef, ...props }: { innerRef?: Ref<Element> }) {
  return <div ref={innerRef} {...props} />;
}
```

### Native Preact Component Patterns

```typescript
import { h } from 'preact';
import { useState, useRef, useEffect } from 'preact/hooks';

// Standard component - no compat needed
export function MyComponent({ children, onClick }: MyComponentProps) {
  const [isOpen, setIsOpen] = useState(false);
  const containerRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    // Side effects
  }, []);

  return (
    <div ref={containerRef} onClick={onClick}>
      {isOpen ? children : null}
    </div>
  );
}
```

### Key Differences from React

| Feature | Preact Native | Don't Use |
|---------|---------------|-----------|
| createElement | `h()` | `React.createElement` |
| refs | Direct ref prop or `useRef` | `forwardRef` from compat |
| Context | `createContext()` from 'preact' | React Context |
| CSS Properties | `CSSProperties` type | No special type needed |

---

## 2. Component Architecture

### Atomic Design Structure

```
src/components/
├── ui/           # Design atoms (Button, Modal, Input, Switch)
├── shared/       # Domain-aware reusable (TokenRow, ChainBadge)
├── features/     # Business logic modules (swap, chart, liquidity)
└── layout/       # App shell (Header, Footer, PageWrapper)
```

**Rules**:
- `ui/` - Pure, stateless, design tokens only. No business logic.
- `shared/` - Reusable across features, domain-aware but feature-agnostic
- `features/` - Heavy business logic, often route-specific
- `layout/` - App structure, navigation containers, page wrappers

### Component Template

```typescript
import { h } from 'preact';
import { useState, useCallback } from 'preact/hooks';

export interface MyComponentProps {
  variant?: 'default' | 'primary';
  children: h.JSX.Element | string;
  className?: string;
  onClick?: () => void;
}

export function MyComponent(props: MyComponentProps) {
  const { variant = 'default', children, className, onClick } = props;

  const handleClick = useCallback(() => {
    onClick?.();
  }, [onClick]);

  return (
    <div
      class={cn('base-classes', variant === 'primary' && 'primary-classes', className)}
      onClick={handleClick}
    >
      {children}
    </div>
  );
}
```

### Key Conventions

| Convention | Example |
|------------|---------|
| Components | PascalCase (`SwapCard`, `PriceChart`) |
| Props interfaces | `ComponentNameProps` |
| Function declarations | `function Component() {}` preferred over const |
| Type imports | `import type { ... }` |
| Barrel exports | Index files in each directory |
| Class prop | Always `class`, never `className` (except when TS type requires) |

---

## 3. Preact Signals

### Signal Store Pattern

```typescript
import { signal, computed, effect } from '@preact/signals';

export class SwapStore {
  // Primitive signals
  public readonly direction = signal<OrderDirection>('sell');
  public readonly orderType = signal<OrderType>('market');

  // Array signals
  public readonly primaryTokens = signal<TokenData[]>([]);

  // Computed values - automatic dependency tracking
  public readonly filteredTokens = computed(() =>
    this.primaryTokens.value.filter(t => t.active)
  );

  // Actions as methods
  public setDirection = (dir: OrderDirection): void => {
    this.direction.value = dir;
  };

  public updateTokens = (tokens: TokenData[]): void => {
    this.primaryTokens.value = tokens;
  };

  // Batch multiple updates
  public batchUpdate = (dir: OrderDirection, tokens: TokenData[]): void => {
    batch(() => {
      this.direction.value = dir;
      this.primaryTokens.value = tokens;
    });
  };
}

// Singleton export - import throughout app
export const swapStore = new SwapStore();
```

### Signal Usage in Components

```typescript
import { swapStore } from './SwapStore';

function SwapInterface() {
  // Direct access - no hook needed
  const direction = swapStore.direction.value;

  const handleDirectionChange = () => {
    swapStore.setDirection('buy');
  };

  return (
    <button onClick={handleDirectionChange}>
      Current: {direction}
    </button>
  );
}
```

### Local Component State with Signals

```typescript
import { useSignal } from '@preact/signals';

function Counter() {
  // Local signal - scoped to component
  const count = useSignal(0);

  return (
    <button onClick={() => count.value++}>
      Count: {count.value}
    </button>
  );
}
```

### Signal Best Practices

| Pattern | Do | Don't |
|---------|-----|-------|
| Global state | Use `signal()` exports | Don't use Context Provider |
| Local state | Use `useSignal()` hook | Don't overuse for simple values |
| Derived state | Use `computed()` | Don't use `useMemo` + signals |
| Batch updates | Use `batch()` | Don't trigger multiple renders |
| Updates | Mutate `.value` directly | Don't create new objects unnecessarily |
| TypeScript | Always type signals | Don't use `any` |

### Signal State Classification

```typescript
// ✅ Global state - module-level signal export
export const theme = signal<'light' | 'dark'>('dark');

// ✅ Local state - useSignal hook
function Form() {
  const inputValue = useSignal('');
}

// ✅ Derived state - computed
const displayValue = computed(() => inputValue.value.toUpperCase());

// ❌ DON'T - Server state in signals
// Use fetch hooks instead
// const remoteData = signal(null);  // WRONG
```

### Effects with Signals

```typescript
import { effect } from '@preact/signals';

// Side effect when signal changes
effect(() => {
  console.log('Theme changed:', theme.value);
  document.documentElement.classList.toggle('dark', theme.value === 'dark');
});

// Cleanup with effect return value
effect(() => {
  const interval = setInterval(() => {
    // tick
  }, 1000);

  return () => clearInterval(interval);
});
```

---

## 4. Tailwind CSS + CVA

### Design System Tokens

```css
/* styles/index.css */
:root {
  /* Semantic colors - use these, not arbitrary hex values */
  --bg-0: #ffffff;
  --bg-1: #f8fafc;
  --fg-0: #0f172a;
  --fg-1: #475569;
  --primary: #3b82f6;
  --success: #22c55e;
  --warning: #f59e0b;
  --error: #ef4444;

  /* Spacing scale - use unit-* for consistency */
  --unit-1: 0.25rem;   /* 4px */
  --unit-2: 0.5rem;    /* 8px */
  --unit-3: 0.75rem;   /* 12px */
  --unit-4: 1rem;      /* 16px */
  --unit-6: 1.5rem;    /* 24px */
  --unit-8: 2rem;      /* 32px */
}
```

### Using CVA for Variants

```typescript
import { cva, type VariantProps } from 'class-variance-authority';

// Define variants with CVA
const buttonVariants = cva(
  // Base classes - always applied
  'inline-flex items-center justify-center rounded-lg font-medium transition-colors focus:outline-none focus:ring-2',
  {
    variants: {
      variant: {
        default: 'bg-muted text-fg-1 hover:bg-muted/80',
        primary: 'bg-primary text-black hover:bg-primary/90',
        danger: 'bg-error text-white hover:bg-error/90',
        ghost: 'hover:bg-muted/50 text-fg-1',
      },
      size: {
        sm: 'h-unit-8 px-unit-3 text-sm',
        md: 'h-unit-10 px-unit-4 text-base',
        lg: 'h-unit-12 px-unit-6 text-lg',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'md',
    },
  }
);

// Export variant type for consumers
export type ButtonVariants = VariantProps<typeof buttonVariants>;

// Usage in component
export function Button({ variant, size, className, ...props }: ButtonProps) {
  return (
    <button
      class={buttonVariants({ variant, size, className })}
      {...props}
    />
  );
}
```

### Tailwind Conventions

| Category | Convention | Example |
|----------|------------|---------|
| Spacing | Use `unit-*` scale | `p-unit-4`, `gap-unit-2` |
| Colors | Use semantic tokens | `bg-bg-0`, `text-fg-1` |
| Responsiveness | Mobile-first | `md:flex`, `lg:w-64` |
| Dark mode | Class-based via theme signal | `dark:bg-gray-900` |
| Components | Use CVA for variants | See above |
| Arbitrary values | Only for one-offs | `w-[137px]` |

### Utility Pattern

```typescript
// utils/cn.ts - className merger
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

// Usage
<div class={cn('base-class', isActive && 'active-class', props.className)} />
```

---

## 5. Custom Router

### Router Structure

```typescript
// lib/router.tsx
import { h, createContext } from 'preact';
import { useState, useEffect } from 'preact/hooks';

interface RouteState {
  path: string;
  query: URLSearchParams;
}

interface RouterContextValue {
  route: RouteState;
  navigate: (to: string) => void;
}

export const RouterContext = createContext<RouterContextValue | null>(null);

export function RouterProvider({ children }: { children: h.JSX.Element | h.JSX.Element[] }) {
  const [route, setRoute] = useState<RouteState>({
    path: window.location.pathname,
    query: new URLSearchParams(window.location.search)
  });

  // Handle browser navigation
  useEffect(() => {
    const handlePopState = () => {
      setRoute({
        path: window.location.pathname,
        query: new URLSearchParams(window.location.search)
      });
    };

    window.addEventListener('popstate', handlePopState);
    return () => window.removeEventListener('popstate', handlePopState);
  }, []);

  const navigate = (to: string): void => {
    window.history.pushState({}, '', to);
    setRoute({
      path: to,
      query: new URLSearchParams(window.location.search)
    });
  };

  return (
    <RouterContext.Provider value={{ route, navigate }}>
      {children}
    </RouterContext.Provider>
  );
}

// Hook for using router
export function useRouter() {
  const context = useContext(RouterContext);
  if (!context) throw new Error('useRouter must be within RouterProvider');
  return context;
}
```

### Navigation Constants

```typescript
// constants/navigation.ts
export interface NavItem {
  path: string;
  label: string;
  icon: string;
}

export const NAVIGATION: readonly NavItem[] = [
  { path: '/', label: 'Swap', icon: 'arrowsLeftRight' },
  { path: '/chart', label: 'Chart', icon: 'chartLineUp' },
  { path: '/liquidity', label: 'Liquidity', icon: 'drops' },
  { path: '/docs', label: 'Docs', icon: 'book' },
] as const;
```

### Routing Best Practices

- Use lazy loading for all pages: `lazy(() => import('./pages/SwapPage'))`
- Define routes in `constants/navigation.ts`
- Use query params for transient state (filters, selections)
- Never store navigation state in signals - router is source of truth
- Keep route definitions in one place

---

## 6. TradingView Lightweight Charts

### Chart Engine Pattern

```typescript
import { createChart, type IChartApi, type ISeriesApi } from 'lightweight-charts';
import { useEffect, useRef } from 'preact/hooks';

export interface ChartConfig {
  height: number;
  chartType?: 'candlestick' | 'line' | 'bar';
  candles: OHLCData[];
  livePrice?: number;
}

export function usePriceChartEngine(config: ChartConfig) {
  const containerRef = useRef<HTMLDivElement>(null);
  const chartRef = useRef<IChartApi | null>(null);
  const seriesRef = useRef<ISeriesApi<'Candlestick'> | null>(null);

  // Initialize chart
  useEffect(() => {
    if (!containerRef.current) return;

    const chart = createChart(containerRef.current, {
      width: containerRef.current.clientWidth,
      height: config.height,
      layout: {
        background: { color: 'transparent' },
        textColor: '#d1d5db',
      },
      grid: {
        vertLines: { color: 'rgba(42, 46, 57, 0.5)' },
        horzLines: { color: 'rgba(42, 46, 57, 0.5)' },
      },
      timeScale: {
        timeVisible: true,
        secondsVisible: false,
        borderColor: '#2b2b43',
      },
      rightPriceScale: {
        borderColor: '#2b2b43',
      },
    });

    chartRef.current = chart;

    // Create series
    const series = chart.addCandlestickSeries({
      upColor: '#26a69a',
      downColor: '#ef5350',
      borderDownColor: '#ef5350',
      borderUpColor: '#26a69a',
      wickDownColor: '#ef5350',
      wickUpColor: '#26a69a',
    });

    seriesRef.current = series;

    // Cleanup on unmount
    return () => {
      chart.remove();
      chartRef.current = null;
      seriesRef.current = null;
    };
  }, [config.height]);

  // Update data when candles change
  useEffect(() => {
    if (!seriesRef.current || !config.candles.length) return;

    seriesRef.current.setData(config.candles);

    // Fit content after data load
    chartRef.current?.timeScale().fitContent();
  }, [config.candles]);

  // Live price updates
  useEffect(() => {
    if (!seriesRef.current || !config.livePrice) return;

    const lastCandle = config.candles[config.candles.length - 1];
    if (!lastCandle) return;

    seriesRef.current.update({
      ...lastCandle,
      close: config.livePrice,
      high: Math.max(lastCandle.high, config.livePrice),
      low: Math.min(lastCandle.low, config.livePrice),
    });
  }, [config.livePrice, config.candles]);

  return { containerRef };
}
```

### Chart Best Practices

| Practice | Description |
|----------|-------------|
| Container ref | Always use ref for container element |
| Cleanup | Always call `chart.remove()` on unmount |
| Resize | Use ResizeObserver to handle container size changes |
| Time scale | Use `timeScale().fitContent()` after data load |
| Real-time | Use `series.update()` for live tick updates |
| Performance | Debounce rapid updates, limit visible candles to ~2000 |
| Memory | Remove series and chart on unmount |

### Responsive Chart

```typescript
function ResponsiveChart({ data }: ChartProps) {
  const containerRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!containerRef.current || !chartRef.current) return;

    const handleResize = () => {
      chartRef.current?.applyOptions({
        width: containerRef.current?.clientWidth,
      });
    };

    const resizeObserver = new ResizeObserver(handleResize);
    resizeObserver.observe(containerRef.current);

    return () => resizeObserver.disconnect();
  }, []);

  return <div ref={containerRef} class="w-full h-full" />;
}
```

---

## 7. Build-Time Compilation

**Zero runtime deps for content: markdown → HTML, AsciiMath → MathML, Mermaid → SVG at build time**

### Markdown Compilation

- **Script**: `scripts/precompile-markdown.ts` (backend only)
- **Process**:
  - Parses markdown with `marked`
  - Syntax highlights with `prismjs` (JS/TS/JSX/TSX/JSON/Bash/SQL/Markdown/Solidity)
  - Converts inline math ($...$) and block math ($$...$$) to MathML using `asciimath2ml`
  - Renders mermaid diagrams to themed SVG (light/dark) via Playwright
  - Generates heading anchors with IDs
- **Output**: `/front/public/compiled-docs/docs.json` (pre-rendered HTML)
- **Frontend**: Uses `MarkdownRenderer` component that loads pre-compiled HTML from JSON (NO parsing)

### Frontend Markdown Usage

```tsx
// Load by slug from pre-compiled docs
<MarkdownRenderer slug="overview" />

// Or pass pre-rendered HTML directly
<MarkdownRenderer content="<p>...</p>" />
```

**Key**: Frontend has ZERO markdown dependencies (`marked`, `prismjs`, `asciimath2ml`, `mermaid` only in backend)

### Search Index

- **Script**: `scripts/build-search-index.ts`
- **What**: Extracts frontmatter, strips markdown, builds full-text search index
- **Output**: Searchable JSON metadata

### Build Chain

```bash
bun run build
# Runs: build:markdown → build:search-index → vite build
```

**Dependencies**:
- ✅ Backend (`package.json`): marked, prismjs, asciimath2ml, playwright, mermaid
- ❌ Frontend (`front/package.json`): NONE of the above (pre-compiled only)

### AsciiMath Syntax Guidelines

**Basic Syntax**:
- Inline: `$expression$`
- Block: `$$expression$$`
- Use `*` for center dot (·), `xx` for cross (×)

**Brackets & Grouping**:
- **Visible parentheses**: `(` `)` — always rendered
- **Invisible grouping**: `{` `}` — grouping only, not rendered
- **Visible curly braces**: `{:` `:}` — rendered as curly braces
- Use invisible grouping for fraction numerators/denominators: `{numerator}/{denominator}`

**Symbol Naming**:
- Prefer single-letter variables: `S`, `U`, `c`
- Use subscripts for variants: `S_v`, `S_f`, `phi_i`, `phi_o`
- Avoid verbose strings: ~~`S_"final"`~~ → `S_f`

**Fractions**:
$$Simple:         a/b$$

$$Complex:        {x + y}/{z - w}$$

$$Multi-term:     {sigma * nu}/{100M}$$

---

## 8. SVG Charts

**SVG, NOT chart.js**

| Chart Type | Status | Location |
|------------|--------|----------|
| Sparklines | ✅ Use SVG | `src/components/charts/` |
| Price data | ✅ TradingView Lightweight | `lightweight-charts` |
| Other charts | 🔲 Build SVG | < 100 lines each |

### SVG Chart Pattern

```tsx
interface SparklineProps {
  data: number[];
  width: number;
  height: number;
  color?: string;
}

export function Sparkline({ data, width, height, color = '#3b82f6' }: SparklineProps) {
  const max = Math.max(...data);
  const min = Math.min(...data);
  const range = max - min || 1;

  const points = data.map((value, i) => {
    const x = (i / (data.length - 1)) * width;
    const y = height - ((value - min) / range) * height;
    return `${x},${y}`;
  }).join(' ');

  return (
    <svg width={width} height={height} viewBox={`0 0 ${width} ${height}`}>
      <polyline
        points={points}
        fill="none"
        stroke={color}
        strokeWidth="2"
      />
    </svg>
  );
}
```

---

## 9. SDK = Source of Truth

**All token/chain/contract metadata in SDK, NOT frontend**

| Data | SDK File |
|------|----------|
| Tokens/addresses/metadata | `sdk/src/eth/tokens.ts` |
| Chain configs | `sdk/src/eth/chains.ts` |
| Contract addresses | `sdk/src/eth/contracts.ts` |

**Rules**: Never duplicate in frontend; always import from `@sdk/eth`

```tsx
// ✅ CORRECT
import { tokens, chains } from '@sdk/eth';

// ❌ WRONG
const TOKENS = { ... };  // Don't duplicate
```

---

## 10. Doc Link Schema

**Format**: `/docs/slug#anchor`

**Slug Generation** (see `sdk/src/utils/format.ts`):
- File: `1.1.1. Inventory Management.md` → Slug: `1.1.1-inventory-management`
- Lowercase, hyphens instead of spaces, preserves section numbers (dots)
- Use `/docs/` prefix for absolute links from anywhere in the docs

**Examples**:
```tsx
// ✅ CORRECT
<a href="/docs/1.1.1-inventory-management#2-coverage-ratio">Inventory</a>
<a href="/docs/1.2.1-core">Core Module</a>

// ❌ WRONG - URL encoding issues
<a href="./1.1.1.%20Inventory%20Management.md">Inventory</a>
<a href="../1.1.1.%20Inventory%20Management.md">Inventory</a>
```

---

## 11. TypeScript Patterns

### Strict Configuration

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "skipLibCheck": true,
    "jsx": "react-jsx",
    "jsxImportSource": "preact",
    "jsxFactory": "h",
    "jsxFragmentFactory": "Fragment",
    "verbatimModuleSyntax": true,
    "esModuleInterop": true,
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@hooks/*": ["./src/hooks/*"],
      "@utils/*": ["./src/utils/*"],
      "@lib/*": ["./src/lib/*"]
    }
  }
}
```

### Type Definitions

```typescript
// Domain types - models/types.ts
export interface TokenData {
  address: string;
  symbol: string;
  decimals: number;
  priceUsd?: number;
  active: boolean;
}

// API types - api/types.ts
export interface SwapQuoteRequest {
  tokenIn: string;
  tokenOut: string;
  amountIn: bigint;
  slippageBps?: number;
}

export interface SwapQuoteResponse {
  amountOut: bigint;
  priceImpact: number;
  gasEstimate: string;
}

// Component props
export interface TokenSelectProps {
  tokens: TokenData[];
  selected?: TokenData;
  onSelect: (token: TokenData) => void;
  class?: string;
}

// Event handlers
export type ClickHandler = (event: MouseEvent) => void;
export type ChangeHandler<T> = (value: T) => void;
```

### Generic Types

```typescript
// Result type for API operations
export type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

// Async state
export type AsyncState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };
```

---

## 12. Performance Patterns

### Code Splitting

```typescript
// Lazy load pages
import { lazy, Suspense } from 'preact/compat';  // NO - don't do this

// ✅ CORRECT - Use dynamic imports with Suspense from Preact
import { Suspense } from 'preact';

const ChartPage = lazy(() => import('./pages/ChartPage'));
const SwapPage = lazy(() => import('./pages/SwapPage'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <ChartPage />
    </Suspense>
  );
}
```

### Memoization

```typescript
import { useMemo, useCallback } from 'preact/hooks';

function ExpensiveComponent({ items, onSelect }: Props) {
  // Memoize expensive computations
  const sortedItems = useMemo(() =>
    [...items].sort((a, b) => a.symbol.localeCompare(b.symbol)),
    [items]
  );

  // Memoize callbacks to prevent child re-renders
  const handleSelect = useCallback((item: Item) => {
    onSelect(item);
  }, [onSelect]);

  return <ItemList items={sortedItems} onSelect={handleSelect} />;
}
```

### Debouncing

```typescript
import { useEffect, useState } from 'preact/hooks';

export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}

// Usage
function SearchInput() {
  const [search, setSearch] = useState('');
  const debouncedSearch = useDebounce(search, 300);

  useEffect(() => {
    // API call with debounced value
    fetchResults(debouncedSearch);
  }, [debouncedSearch]);
}
```

---

## 13. Testing

```typescript
import { render, screen } from '@testing-library/preact';
import { vi } from 'vitest';

describe('SwapButton', () => {
  it('renders with correct label', () => {
    render(<SwapButton label="Swap" />);
    expect(screen.getByText('Swap')).toBeInTheDocument();
  });

  it('calls onClick when clicked', async () => {
    const handleClick = vi.fn();
    const { getByRole } = render(<SwapButton label="Swap" onClick={handleClick} />);

    const button = getByRole('button');
    button.click();

    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});

// Signal testing
import { signal } from '@preact/signals';

describe('SwapStore', () => {
  it('updates direction', () => {
    const store = new SwapStore();
    expect(store.direction.value).toBe('sell');

    store.setDirection('buy');
    expect(store.direction.value).toBe('buy');
  });
});
```

---

## 14. Development Workflow

```bash
# Install dependencies
bun install

# Development with hot reload
bun run dev

# Type checking
bun run typecheck

# Build for production
bun run build

# Run tests
bun test

# Linting
bun run lint
```

---

## 15. Common Patterns

### Modal Pattern

```typescript
import { BaseModal } from '@/components/ui/BaseModal';
import { ModalHeader } from '@/components/ui/ModalHeader';

interface ModalProps {
  open: boolean;
  onClose: () => void;
  title: string;
}

export function MyModal({ open, onClose, title, children }: ModalProps) {
  return (
    <BaseModal open={open} onClose={onClose}>
      <ModalHeader title={title} onClose={onClose} />
      <div class="p-4">
        {children}
      </div>
    </BaseModal>
  );
}
```

### Loading States

```typescript
type AsyncState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };

function DataComponent({ state }: { state: AsyncState<Data> }) {
  switch (state.status) {
    case 'loading':
      return <LoadingSpinner />;
    case 'error':
      return <ErrorMessage message={state.error} />;
    case 'success':
      return <DataDisplay data={state.data} />;
    default:
      return <EmptyState />;
  }
}
```

### Form Handling

```typescript
function Form() {
  const formData = useSignal<FormData>({
    email: '',
    password: '',
  });

  const handleSubmit = (e: JSX.TargetedEvent<HTMLFormElement>) => {
    e.preventDefault();
    submitForm(formData.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={formData.value.email}
        onInput={(e) => formData.value = {
          ...formData.value,
          email: e.currentTarget.value
        }}
      />
    </form>
  );
}
```

---

## 16. File Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase | `SwapCard.tsx` |
| Hooks | camelCase with `use` prefix | `useSwap.ts` |
| Utils | camelCase | `formatPrice.ts` |
| Types | PascalCase `.types.ts` | `pool.types.ts` |
| Constants | PascalCase | `navigation.ts` |
| Styles | kebab-case | `markdown.css` |
| Tests | Same as file + `.test.ts` | `swap.test.ts` |

---

## 17. Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Importing from 'react' | Use 'preact' instead |
| Using preact/compat | Remove, use native Preact |
| `className` in JSX | Use `class` (works with JSX config) |
| Forgetting `.value` on signals | Always access `.value` |
| Context for global state | Use signals instead |
| Server state in signals | Use fetch hooks |
| Arbitrary Tailwind values | Use design tokens |
| Uncontrolled re-renders | Use `useMemo`, `useCallback` |

---

## 18. Context Providers Pattern

```typescript
// lib/wallet.tsx
import { h, createContext } from 'preact';
import { useContext } from 'preact/hooks';

interface WalletContextValue {
  address: string | null;
  connect: () => Promise<void>;
  disconnect: () => void;
}

export const WalletContext = createContext<WalletContextValue>({
  address: null,
  connect: async () => {},
  disconnect: () => {},
});

export function WalletProvider({ children }: { children: h.JSX.Element | h.JSX.Element[] }) {
  const [address, setAddress] = useState<string | null>(null);

  const connect = async (): Promise<void> => {
    // Connection logic
    setAddress('0x...');
  };

  const disconnect = (): void => {
    setAddress(null);
  };

  return (
    <WalletContext.Provider value={{ address, connect, disconnect }}>
      {children}
    </WalletContext.Provider>
  );
}

export function useWallet() {
  return useContext(WalletContext);
}
```

---

## Internal References

- [`CONTRIBUTING.md`](../CONTRIBUTING.md) - Commit conventions and PR guidelines
- [`AGENTS.md`](../AGENTS.md) - AI agent workflows

---

*Last updated: 2026-01*
