---
title: "Back-End Developer Best Practices"
description: "## Quick Reference"
audience: tech
type: reference
status: planned
phase: 2
order: 99
lang: en
publish: true
---
# Back-End Developer Best Practices

**Tech Stack**: Bun + TypeScript + SQLite + Bash/Zig Scripts + Docker + Caddy

---

## Quick Reference

| Area | Standard |
|------|----------|
| **Package Manager** | `bun` exclusively (never npm/yarn) |
| **Runtime** | Bun (not Node.js) |
| **Database** | SQLite via `bun:sqlite` |
| **Language** | TypeScript with strict mode |
| **Scripts** | Bash for automation, Zig for performance-critical |
| **Containers** | Docker multi-stage builds |
| **Reverse Proxy** | Caddy (automatic HTTPS) |

---

## 1. Bun Runtime Patterns

### Shebang Executable Scripts

```typescript
#!/usr/bin/env bun

// server-main.ts - Direct execution
import { serve } from 'bun';

console.log('Starting server...');
```

```bash
# Make executable
chmod +x src/server-main.ts

# Run directly
./src/server-main.ts
```

### Native Bun APIs

```typescript
// HTTP Server
Bun.serve({
  port: 3000,
  fetch(req) {
    return new Response('Hello', { status: 200 });
  },
  websocket: {
    message(ws, message) {
      ws.send(message);
    },
  },
});

// File I/O
const file = Bun.file('./data.json');
const contents = await file.text();

// SQLite
import { Database } from 'bun:sqlite';
const db = new Database('./data.db');
```

### Environment Variables

```typescript
// Reading env vars
const PORT = process.env.PORT ?? '3000';
const DATABASE = process.env.DATABASE_URL ?? './data.db';

// Required env helper
function getEnv(key: string, fallback?: string): string {
  const value = process.env[key];
  if (value === undefined) {
    if (fallback !== undefined) return fallback;
    throw new Error(`Missing required env var: ${key}`);
  }
  return value;
}
```

### Graceful Shutdown

```typescript
const server = Bun.serve({ /* ... */ });

const shutdown = (signal: string) => {
  console.log(`Received ${signal}, shutting down...`);
  server.stop();
  process.exit(0);
};

process.on('SIGINT', () => shutdown('SIGINT'));
process.on('SIGTERM', () => shutdown('SIGTERM'));
```

---

## 2. TypeScript Configuration

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "lib": ["ESNext"],
    "moduleResolution": "bundler",
    "strict": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "verbatimModuleSyntax": true,
    "resolveJsonModule": true,
    "types": ["bun-types"]
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

### Type Patterns

```typescript
// Request/Response types
interface RequestWithUser extends Request {
  userId?: string;
}

// WebSocket client state
interface WSClient {
  id: string;
  subs: Set<string>;
  lastPing: number;
}

// Configuration types
interface ServerConfig {
  port: number;
  databasePath: string;
  corsOrigins: string[];
  redisUrl?: string;
}

// API response wrapper
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
  timestamp: number;
}
```

### Error Handling Types

```typescript
class AppError extends Error {
  constructor(
    public statusCode: number,
    public code: string,
    message: string
  ) {
    super(message);
    this.name = 'AppError';
  }
}

class ValidationError extends AppError {
  constructor(message: string) {
    super(400, 'VALIDATION_ERROR', message);
  }
}

class NotFoundError extends AppError {
  constructor(resource: string) {
    super(404, 'NOT_FOUND', `${resource} not found`);
  }
}
```

---

## 3. SQLite Database Patterns

### Database Setup

```typescript
import { Database } from 'bun:sqlite';

let dbInstance: Database | null = null;

export function getDatabase(): Database {
  if (!dbInstance) {
    dbInstance = new Database('./data/app.db', { create: true });

    // Enable WAL mode for concurrency
    dbInstance.exec('PRAGMA journal_mode=WAL');
    dbInstance.exec('PRAGMA synchronous=NORMAL');
    dbInstance.exec('PRAGMA cache_size=-64000'); // 64MB cache
    dbInstance.exec('PRAGMA temp_store=memory');
  }
  return dbInstance;
}

// Close on shutdown
process.on('beforeExit', () => dbInstance?.close());
```

### Schema Migration Pattern

```typescript
// migrations/migrate.ts
const migrations: string[] = [
  `CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE NOT NULL,
    created_at INTEGER NOT NULL
  )`,
  `CREATE INDEX IF NOT EXISTS idx_users_email ON users(email)`,
  // ... more migrations
];

export function runMigrations(db: Database): void {
  // Track applied migrations
  db.exec(`
    CREATE TABLE IF NOT EXISTS _migrations (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      version INTEGER NOT NULL UNIQUE,
      applied_at INTEGER NOT NULL
    )
  `);

  const applied = db.query<[{ version: number }]>(
    'SELECT version FROM _migrations ORDER BY version'
  ).all();

  const appliedSet = new Set(applied.map(r => r.version));

  for (let i = 0; i < migrations.length; i++) {
    if (!appliedSet.has(i)) {
      db.transaction(() => {
        db.exec(migrations[i]);
        db.query('INSERT INTO _migrations (version, applied_at) VALUES (?, ?)')
          .run(i, Date.now());
      })();
    }
  }
}
```

### Query Patterns

```typescript
// Prepared statements (recommended for performance)
interface CandleRow {
  pair: string;
  timestamp: number;
  open: number;
  high: number;
  low: number;
  close: number;
  volume: number;
}

class CandleStore {
  private insertStmt: Statement;
  private selectStmt: Statement;

  constructor(private db: Database) {
    this.insertStmt = db.query(`
      INSERT OR REPLACE INTO candles (pair, timestamp, open, high, low, close, volume)
      VALUES (?, ?, ?, ?, ?, ?, ?)
    `);

    this.selectStmt = db.query<CandleRow, [string, number, number]>(`
      SELECT pair, timestamp, open, high, low, close, volume
      FROM candles
      WHERE pair = ? AND timestamp >= ? AND timestamp <= ?
      ORDER BY timestamp ASC
    `);
  }

  insert(candle: CandleRow): void {
    this.insertStmt.run(
      candle.pair,
      candle.timestamp,
      candle.open,
      candle.high,
      candle.low,
      candle.close,
      candle.volume
    );
  }

  getRange(pair: string, from: number, to: number): CandleRow[] {
    return this.selectStmt.all(pair, from, to);
  }
}
```

### Transaction Pattern

```typescript
import type { Transaction } from 'bun:sqlite';

// Using transaction for atomic operations
function transfer(db: Database, fromId: number, toId: number, amount: number): boolean {
  return db.transaction(() => {
    const balance = db.query<{ balance: number }, [number]>(
      'SELECT balance FROM accounts WHERE id = ? FOR UPDATE'
    ).get(fromId);

    if (!balance || balance.balance < amount) {
      return false;
    }

    db.query('UPDATE accounts SET balance = balance - ? WHERE id = ?').run(amount, fromId);
    db.query('UPDATE accounts SET balance = balance + ? WHERE id = ?').run(amount, toId);

    return true;
  })();
}
```

---

## 4. HTTP Server Patterns

### REST API Structure

```typescript
import { serve } from 'bun';
import { Router } from './router';

const router = new Router();

// Define routes
router.get('/api/health', async (req) => {
  return Response.json({ status: 'ok', timestamp: Date.now() });
});

router.get('/api/price/:symbol', async (req, params) => {
  const { symbol } = params;
  const price = await getPrice(symbol);
  return Response.json({ symbol, price });
});

router.post('/api/quote', async (req) => {
  const body = await req.json();
  const quote = await calculateQuote(body);
  return Response.json(quote);
});

// Start server
serve({
  port: 3000,
  fetch: router.fetch.bind(router),
  error(error) {
    console.error('Server error:', error);
    return new Response('Internal Error', { status: 500 });
  },
});
```

### Router Implementation

```typescript
type RouteHandler = (req: Request, params: Record<string, string>) => Promise<Response> | Response;

interface Route {
  method: string;
  pattern: URLPattern;
  handler: RouteHandler;
}

export class Router {
  private routes: Route[] = [];

  add(method: string, path: string, handler: RouteHandler): void {
    this.routes.push({
      method,
      pattern: new URLPattern({ pathname: path }),
      handler,
    });
  }

  get(path: string, handler: RouteHandler): void {
    this.add('GET', path, handler);
  }

  post(path: string, handler: RouteHandler): void {
    this.add('POST', path, handler);
  }

  async fetch(req: Request): Promise<Response> {
    const url = new URL(req.url);
    const method = req.method;

    for (const route of this.routes) {
      if (route.method !== method) continue;

      const match = route.pattern.exec(url);
      if (match) {
        const params = match.pathname.groups;
        try {
          return await route.handler(req, params);
        } catch (error) {
          return this.handleError(error);
        }
      }
    }

    return new Response('Not Found', { status: 404 });
  }

  private handleError(error: unknown): Response {
    if (error instanceof AppError) {
      return Response.json({
        success: false,
        error: error.message,
        code: error.code,
      }, { status: error.statusCode });
    }
    return Response.json({
      success: false,
      error: 'Internal server error',
    }, { status: 500 });
  }
}
```

### CORS Pattern

```typescript
function cors(req: Request): Response {
  const origin = req.headers.get('Origin');
  const allowedOrigins = process.env.CORS_ORIGINS?.split(',') ?? ['http://localhost:3000'];

  const headers = new Headers();
  if (origin && allowedOrigins.includes(origin)) {
    headers.set('Access-Control-Allow-Origin', origin);
    headers.set('Access-Control-Allow-Methods', 'GET, POST, OPTIONS');
    headers.set('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  }

  // Handle preflight
  if (req.method === 'OPTIONS') {
    return new Response(null, { headers });
  }

  return new Response(null, { headers }); // Add to actual response
}
```

---

## 5. WebSocket Patterns

### WebSocket Server

```typescript
interface WSClient {
  id: string;
  subs: Set<string>;
  send: (data: string) => void;
}

const clients = new Map<string, WSClient>();

Bun.serve({
  port: 3001,
  fetch(req, server) {
    const upgrade = req.headers.get('upgrade') ?? '';
    if (upgrade.toLowerCase() !== 'websocket') {
      return new Response('Expected WebSocket', { status: 426 });
    }

    server.upgrade(req);
    return new Response(); // Upgraded
  },
  websocket: {
    open(ws) {
      const clientId = crypto.randomUUID();
      const client: WSClient = {
        id: clientId,
        subs: new Set(),
        send: (data) => ws.send(data),
      };
      clients.set(clientId, client);
      console.log(`Client connected: ${clientId}`);
    },
    message(ws, message) {
      const data = JSON.parse(message.toString());

      // Handle subscription
      if (data.type === 'subscribe') {
        const client = clients.get(ws.data.id);
        client?.subs.add(data.symbol);
      }
    },
    close(ws) {
      clients.delete(ws.data.id);
      console.log(`Client disconnected: ${ws.data.id}`);
    },
  },
});

// Broadcast to subscribers
function broadcast(symbol: string, data: unknown): void {
  const payload = JSON.stringify({ type: 'price', symbol, data });
  for (const client of clients.values()) {
    if (client.subs.has(symbol)) {
      client.send(payload);
    }
  }
}
```

---

## 6. File Organization

### Directory Structure

```
back/
├── collector/              # Service name
│   ├── src/
│   │   ├── server-main.ts  # Entry point with shebang
│   │   ├── server.ts       # HTTP/WebSocket server
│   │   ├── collector.ts    # Business logic
│   │   ├── storage.ts      # Database layer
│   │   ├── config.ts       # Configuration
│   │   ├── types.ts        # Type definitions
│   │   └── utils/          # Utilities
│   ├── data/               # SQLite data (gitignored)
│   ├── package.json
│   ├── tsconfig.json
│   ├── Dockerfile
│   └── build.sh
```

### Module Structure

```typescript
// Export patterns
export { default } from './server';      // Default export
export { Server } from './server';       // Named export
export type { ServerConfig } from './server'; // Type-only export
```

---

## 7. Docker Patterns

### Multi-stage Dockerfile

```dockerfile
# Build stage
FROM oven/bun:1 AS build
WORKDIR /app

# Install dependencies
COPY package.json bun.lock* ./
RUN bun install --frozen-lockfile --production

# Copy source
COPY src ./src

# Production stage
FROM oven/bun:1-base
WORKDIR /app

# Copy from build stage
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/src ./src
COPY --from=build /app/package.json ./

# Create non-root user
RUN useradd -m -u 1001 appuser && \
    chown -R appuser:appuser /app
USER appuser

# Data volume
VOLUME ["/app/data"]

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

EXPOSE 3000
CMD ["bun", "run", "src/server-main.ts"]
```

### docker-compose.yml

```yaml
version: '3.8'
services:
  collector:
    build: .
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - ./data:/app/data
    environment:
      - NODE_ENV=production
      - DATABASE_PATH=/app/data/app.db
    healthcheck:
      test: ["CMD", "wget", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 3s
      retries: 3

  caddy:
    image: caddy:latest
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config
    depends_on:
      - collector

volumes:
  caddy_data:
  caddy_config:
```

---

## 8. Caddy Reverse Proxy

### Caddyfile

```caddyfile
# Global options
{
  email admin@example.com
}

# Main domain
example.com {
  # Reverse proxy to backend
  reverse_proxy collector:3000

  # WebSocket support
  header_up X-Real-IP {remote_host}
  header_up X-Forwarded-For {remote_host}
  header_up X-Forwarded-Proto {scheme}

  # Logging
  log {
    output file /var/log/caddy/access.log
  }
}

# WebSocket endpoint
ws.example.com {
  reverse_proxy collector:3001
}

# API with rate limiting
api.example.com {
  reverse_proxy collector:3000

  # Basic rate limiting (requires plugin)
  rate_limit {
    zone api_limit {
      key {remote_host}
      events 100
      window 1m
    }
  }
}
```

### Caddy Best Practices

| Pattern | Description |
|---------|-------------|
| Automatic HTTPS | Caddy handles certificates automatically via Let's Encrypt |
| WebSocket | No special config needed - Caddy handles upgrade headers |
| Static files | Use `file_server` directive |
| Headers | Use `header_up` for upstream, `header_down` for response |
| Logging | JSON format recommended for log aggregation |
| Health checks | Use `/health` endpoint for container health |

---

## 9. Bash Scripts

### Build Script Template

```bash
#!/usr/bin/env bash
set -euo pipefail

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

log_info() {
  echo -e "${GREEN}[INFO]${NC} $1"
}

log_warn() {
  echo -e "${YELLOW}[WARN]${NC} $1"
}

log_error() {
  echo -e "${RED}[ERROR]${NC} $1"
  exit 1
}

# Check dependencies
command -v bun >/dev/null 2>&1 || log_error "Bun not installed"
command -v docker >/dev/null 2>&1 || log_error "Docker not installed"

# Main logic
log_info "Building project..."
bun run build

log_info "Building Docker image..."
docker build -t myapp:latest .

log_info "Done!"
```

### Deployment Script

```bash
#!/usr/bin/env bash
set -euo pipefail

SERVICE_NAME="collector"
IMAGE_NAME="myapp"
REGISTRY="registry.example.com"

# Build and push
log_info "Building image..."
docker build -t ${IMAGE_NAME}:latest .

log_info "Tagging image..."
docker tag ${IMAGE_NAME}:latest ${REGISTRY}/${IMAGE_NAME}:${VERSION}

log_info "Pushing to registry..."
docker push ${REGISTRY}/${IMAGE_NAME}:${VERSION}

# Deploy (kubernetes example)
log_info "Deploying..."
kubectl set image deployment/${SERVICE_NAME} \
  ${SERVICE_NAME}=${REGISTRY}/${IMAGE_NAME}:${VERSION}

log_info "Waiting for rollout..."
kubectl rollout status deployment/${SERVICE_NAME}
```

---

## 10. Zig for Performance-Critical Code

### When to Use Zig

| Use Case | Example |
|----------|---------|
| Hot path computation | Pricing calculations, Monte Carlo simulation |
| Data processing | Large dataset transformations |
| FFI bridges | High-performance library bindings |
| CLI tools | Build/deployment utilities |

### Zig-Bun Interop

```zig
// math.zig - Export as WASM
const std = @import("std");

export fn calculatePrice(base: f64, quote: f64, amount: f64) f64 {
    return (quote / base) * amount;
}

// Compile to WASM
// zig build-lib math.zig -target wasm32-wasi -dynamic -OReleaseFast
```

```typescript
// Load WASM module in Bun
const mathWasm = await Bun.file('./math.wasm').arrayBuffer();
const mathModule = await WebAssembly.instantiate(mathWasm);
const { calculatePrice } = mathModule.instance.exports;

const price = calculatePrice(1.0, 2.5, 100);
```

---

## 11. Logging Patterns

```typescript
// Structured logging
type LogLevel = 'debug' | 'info' | 'warn' | 'error';

interface LogEntry {
  level: LogLevel;
  message: string;
  timestamp: string;
  context?: Record<string, unknown>;
}

class Logger {
  private context: Record<string, unknown>;

  constructor(context: Record<string, unknown> = {}) {
    this.context = context;
  }

  private log(level: LogLevel, message: string, extra?: Record<string, unknown>): void {
    const entry: LogEntry = {
      level,
      message,
      timestamp: new Date().toISOString(),
      context: { ...this.context, ...extra },
    };

    const output = JSON.stringify(entry);
    switch (level) {
      case 'error': console.error(output); break;
      case 'warn': console.warn(output); break;
      case 'info': console.log(output); break;
      case 'debug': process.env.DEBUG && console.debug(output); break;
    }
  }

  debug(message: string, extra?: Record<string, unknown>): void {
    this.log('debug', message, extra);
  }

  info(message: string, extra?: Record<string, unknown>): void {
    this.log('info', message, extra);
  }

  warn(message: string, extra?: Record<string, unknown>): void {
    this.log('warn', message, extra);
  }

  error(message: string, extra?: Record<string, unknown>): void {
    this.log('error', message, extra);
  }

  with(context: Record<string, unknown>): Logger {
    return new Logger({ ...this.context, ...context });
  }
}

// Usage
const log = new Logger({ service: 'collector' });
log.info('Server started', { port: 3000 });
```

---

## 12. Testing

```typescript
import { describe, it, expect, beforeAll, afterAll } from 'bun:test';

describe('CandleStore', () => {
  let db: Database;
  let store: CandleStore;

  beforeAll(() => {
    db = new Database(':memory:');
    runMigrations(db);
    store = new CandleStore(db);
  });

  afterAll(() => {
    db.close();
  });

  it('should store and retrieve candles', () => {
    const candle: CandleRow = {
      pair: 'BTCUSDT',
      timestamp: 1234567890,
      open: 50000,
      high: 51000,
      low: 49000,
      close: 50500,
      volume: 100,
    };

    store.insert(candle);
    const retrieved = store.getRange('BTCUSDT', 1234567800, 1234568000);

    expect(retrieved).toHaveLength(1);
    expect(retrieved[0].close).toBe(50500);
  });
});
```

---

## 13. Development Workflow

```bash
# Install dependencies
bun install

# Development with watch
bun run dev

# Type checking
bun run typecheck

# Linting
bun run lint
bun run lint --fix

# Run tests
bun test

# Production build
bun run build

# Run in production
bun run start
```

---

## 14. Package.json Scripts

```json
{
  "name": "my-service",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "bun --watch src/server-main.ts",
    "start": "bun src/server-main.ts",
    "build": "bun build src/server-main.ts --outdir ./dist --target bun",
    "typecheck": "bunx tsc --noEmit",
    "lint": "oxlint src",
    "fmt": "oxlint --fix src",
    "test": "bun test",
    "test:watch": "bun test --watch",
    "docker:build": "docker build -t myapp .",
    "docker:run": "docker run -p 3000:3000 myapp"
  },
  "dependencies": {
    "@types/bun": "latest"
  },
  "devDependencies": {
    "oxlint": "^0.1.0"
  }
}
```

---

## 15. Common Patterns

### Singleton Pattern

```typescript
class DatabaseManager {
  private static instance: DatabaseManager;
  private db: Database;

  private constructor() {
    this.db = new Database('./data.db');
  }

  static getInstance(): DatabaseManager {
    if (!DatabaseManager.instance) {
      DatabaseManager.instance = new DatabaseManager();
    }
    return DatabaseManager.instance;
  }

  query(sql: string, params: unknown[]) {
    return this.db.query(sql).all(...params);
  }
}

// Usage
const db = DatabaseManager.getInstance();
```

### Event Emitter Pattern

```typescript
type Listener<T> = (data: T) => void;

class EventEmitter<T> {
  private listeners = new Set<Listener<T>>();

  on(listener: Listener<T>): () => void {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  }

  emit(data: T): void {
    for (const listener of this.listeners) {
      try {
        listener(data);
      } catch (error) {
        console.error('Listener error:', error);
      }
    }
  }
}

// Usage
const priceEmitter = new EventEmitter<PriceUpdate>();
priceEmitter.on((price) => console.log('Price:', price));
priceEmitter.emit({ symbol: 'BTC', price: 50000 });
```

---

## 16. Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Using Node.js APIs | Use Bun native APIs (`Bun.file`, `Bun.serve`) |
| Forgetting SQLite WAL | Always enable WAL mode for concurrency |
| Blocking event loop | Offload heavy work to worker threads or Zig |
| Not cleaning up resources | Always close DB connections, clear timeouts |
| Ignoring errors | Always handle promise rejections |
| Hardcoded config | Use environment variables |
| Missing health checks | Provide `/health` endpoint |
| No graceful shutdown | Handle SIGTERM/SIGINT |

---

## 17. Performance Checklist

- [ ] Enable SQLite WAL mode
- [ ] Use prepared statements for repeated queries
- [ ] Add database indexes for common query patterns
- [ ] Use `bun:sqlite` instead of external SQLite libraries
- [ ] Implement connection pooling for external services
- [ ] Cache expensive computations
- [ ] Use streaming for large file operations
- [ ] Profile with `bun --prof` for hot paths

---

## Internal References

- [`GIT.md`](./GIT.md) - Commit conventions
- [`FRONTEND.md`](./FRONTEND.md) - Front-end best practices

---

*Last updated: 2025-01*
