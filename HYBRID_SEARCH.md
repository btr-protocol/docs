# Hybrid Search System: Technical Specification

**For external review | Version 1.0 | 2025-01**

## Executive Summary

The hybrid search system combines **BM25 lexical search** and **vector semantic search** using **Reciprocal Rank Fusion (RRF)**. When both methods identify the same source file, the system merges the line ranges, combines content, and applies a 10% relevance boost.

---

## Table of Contents

1. [System Architecture](#1-system-architecture)
2. [Query Processing Pipeline](#2-query-processing-pipeline)
3. [Lexical Search (BM25)](#3-lexical-search-bm25)
4. [Semantic Search (RAG)](#4-semantic-search-rag)
5. [RRF Fusion & Source Merging](#5-rrf-fusion--source-merging)
6. [Scoring & Ranking](#6-scoring--ranking)
7. [Context Formatting](#7-context-formatting)
8. [Performance Characteristics](#8-performance-characteristics)
9. [Configuration Reference](#9-configuration-reference)

---

## 1. System Architecture

### 1.1 Component Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              User Query                                 │
└─────────────────────────────────────┬───────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         Query Optimizer                                 │
│  • LLM-powered rephrasing                                              │
│  • Synonym extraction                                                 │
│  • Related term discovery                                             │
└─────────────────────────────────────┬───────────────────────────────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
                    ▼                           ▼
┌─────────────────────────────┐  ┌─────────────────────────────┐
│      Lexical Search          │  │      Semantic Search         │
│      (BM25 Algorithm)         │  │      (Vector RAG)            │
│                             │  │                             │
│  • Tokenization              │  │  • TEI Embeddings            │
│  • Stemming                  │  │  • LanceDB (768D)            │
│  • Fuzzy matching            │  │  • Cosine similarity         │
│  • IDF calculation           │  │  • Top-k retrieval           │
└─────────────┬───────────────┘  └─────────────┬───────────────┘
              │                               │
              │     Ranked Results            │
              │     (score + rank)            │
              └───────────┬───────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                       RRF Fusion Engine                                  │
│                                                                         │
│  1. Group results by file path                                        │
│  2. Detect overlaps (same file from both methods)                     │
│  3. Merge line ranges (min-start to max-end)                          │
│  4. Combine content with separator                                     │
│  5. Apply hybrid boost: max(scores) × 1.1                             │
│  6. Mark origin: lexical | semantic | hybrid                          │
└─────────────────────────────────────┬───────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        Ranking & Filtering                               │
│  • Sort by compound score (descending)                                │
│  • Apply max results limit                                            │
│  • Filter by min similarity threshold                                 │
└─────────────────────────────────────┬───────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                       Context Formatting                                │
│  • Prioritize: hybrid > semantic > lexical                           │
│  • Format as markdown with source headers                             │
│  • Truncate to fit context budget                                     │
└─────────────────────────────────────┬───────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          LLM Response                                   │
│  • Receives verbatim extracts as context                              │
│  • Generates answer citing sources                                    │
└─────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Data Flow

```
Query → Optimizer → [Lexical‖Semantic] → Ranked Lists → RRF Fusion → Merged Results → Context → LLM
```

---

## 2. Query Processing Pipeline

### 2.1 Query Optimization (Optional)

Before searching, queries are processed by an LLM to improve retrieval:

| Input | Output | Purpose |
|-------|--------|---------|
| "pricng" | "pricing mechanism fees" | Typos → correct terms |
| "How does AIMM work?" | "AIMM automated market maker anchor path" | Add technical terms |

**Output Structure:**

```typescript
interface OptimizedQuery {
  queryType: 'technical' | 'general';
  rephrasedQuery: string;        // Clearer version
  synonyms: string[];            // e.g., ["fees", "cost"]
  relatedTerms: string[];        // e.g., ["pool", "inventory"]
  expandedSearchString: string;  // For BM25: "pricing fees cost pool..."
}
```

**Can be skipped** via `skipQueryOptimization: true` option.

### 2.2 Search Execution

After optimization (or with original query), both searches run **in parallel**:

```typescript
const [lexicalResults, semanticResults] = await Promise.all([
  lexicalSearch(searchString),
  semanticSearch(searchString)
]);
```

---

## 3. Lexical Search (BM25)

### 3.1 Algorithm

**BM25 with Robertson-Sparck Jones IDF:**

```
score(D,Q) = Σ IDF(qi) × (f(qi,D) × (k1 + 1)) / (f(qi,D) + k1 × (1 - b + b × |D|/avgDl))
```

**Where:**
- `D` = document
- `Q` = query (set of terms)
- `f(qi,D)` = frequency of term qi in document D
- `|D|` = document length (tokens)
- `avgDl` = average document length
- `k1` = 1.2 (term frequency saturation)
- `b` = 0.75 (length normalization)

### 3.2 Text Processing

**Tokenization:**
- camelCase → camel case
- snake_case → snake case
- kebab-case → kebab case
- Preserves technical terms: EIP-20, ERC721, USDC

**Stemming:**
- Removes suffixes: ing, ly, ed, es, s, ment, tion, ation, ize
- Preserves technical acronyms: AMM, CLMM, LVR, TWAP, VWAP, ETH

**Fuzzy Matching:**
- Levenshtein distance for typo tolerance
- Max normalized distance: 0.2
- Minimum word length: 4 characters

### 3.3 Output Format

```typescript
SearchResult {
  content: string;              // Exact text from source file
  origin: 'lexical';
  location: {
    file: string;               // e.g., "docs/Pricing.md:10-25"
    language: string;           // e.g., "markdown", "solidity"
    type: 'code' | 'docs';
    section: string;            // e.g., "section.subsection"
  };
  scores: {
    type: 'lexical';
    lexicalScore: number;       // BM25 normalized to 0-1
    lexicalRank: number;        // 1-indexed position
    lexicalMatchCount: number;  // Number of query term matches
    compound: number;           // RRF contribution (not final score)
  };
}
```

---

## 4. Semantic Search (RAG)

### 4.1 Embedding Model

| Property | Value |
|----------|-------|
| Model | `onnx-community/embeddinggemma-300m-ONNX` |
| Dimensions | 768 |
| Chunk size | 840 characters |
| Overlap | 160 characters (19%) |
| Provider | TEI (Text Embeddings Inference) |

### 4.2 Query Prefixing

EmbeddingGemma uses instruction-based prefixes:

```
Query:   "query: how does pricing work?"  ← Added prefix
Document: "the pricing mechanism is..."    ← No prefix
```

### 4.3 Similarity Calculation

```
similarity = cosine_distance(query_vector, chunk_vector)
           = 1 - (query · chunk) / (||query|| × ||chunk||)
```

Range: 0 (dissimilar) to 1 (identical)

### 4.4 Vector Database

**LanceDB Schema:**

```typescript
{
  id: number;
  vector: number[768];       // Embedding
  text: string;              // Original chunk text
  language: string;          // e.g., "solidity", "typescript"
  type: 'code' | 'docs';
  file: string;              // File path with line range
  section: string;           // Section identifier
  fileDigest: string;        // SHA256 of source file
  indexedAt: string;         // ISO timestamp
}
```

### 4.5 Output Format

```typescript
SearchResult {
  content: string;              // Exact text from source file
  origin: 'semantic';
  location: { /* same as lexical */ };
  scores: {
    type: 'semantic';
    semanticScore: number;     // Cosine similarity (0-1)
    semanticRank: number;      // 1-indexed position
    compound: number;          // RRF contribution
  };
}
```

---

## 5. RRF Fusion & Source Merging

### 5.1 Reciprocal Rank Fusion

**Formula:**

```
RRF_score(d) = Σ 1 / (k + rank_i(d))

where:
  k = 60 (damping constant)
  rank_i(d) = position of document d in ranked list i
```

**Example:**

| Document | Lexical Rank | Semantic Rank | RRF Score |
|----------|--------------|---------------|-----------|
| docs/A.md | 3 | 5 | 1/(60+3) + 1/(60+5) = 0.0303 |
| docs/B.md | 1 | -| 1/(60+1) = 0.0164 |
| docs/C.md | -| 2 | 1/(60+2) = 0.0161 |

### 5.2 Source Merging Algorithm

**Key:** Uses file path only (not section) to detect overlaps.

```typescript
function getKey(location: SearchResultLocation): string {
  return location.file;  // e.g., "docs/Pricing.md:10-25"
}
```

**When same file found by both methods:**

1. **Parse line ranges:**
   ```typescript
   // "docs/Pricing.md:10-17" → { start: 10, end: 17 }
   // "docs/Pricing.md:14-23" → { start: 14, end: 23 }
   ```

2. **Merge ranges:**
   ```typescript
   mergedRange = {
     start: min(10, 14) = 10,
     end: max(17, 23) = 23
   }
   // Result: "docs/Pricing.md:10-23"
   ```

3. **Merge content:**
   ```typescript
   mergedContent = lexicalContent + '\n...\n' + semanticContent
   ```

4. **Boost score:**
   ```typescript
   baseScore = max(lexicalScore, semanticScore)
   boostedScore = min(baseScore × 1.1, 1.0)
   ```

### 5.3 Origin Classification

| Found By | Origin Value | Score Treatment |
|----------|--------------|-----------------|
| Lexical only | `'lexical'` | No boost |
| Semantic only | `'semantic'` | No boost |
| **Both** | **`'hybrid'`** | **×1.1 boost (capped at 1.0)** |

### 5.4 Fusion Pseudocode

```
fusionMap = Map<filePath, SearchResult>()

// 1. Process lexical results
for each lexicalResult in lexicalResults:
  key = lexicalResult.location.file
  rrfScore = 1 / (60 + lexicalRank)
  fusionMap.set(key, {
    content: lexicalResult.content,
    location: lexicalResult.location,
    scores: { type: 'lexical', compound: rrfScore, ... }
  })

// 2. Process semantic results
for each semanticResult in semanticResults:
  key = semanticResult.location.file

  if fusionMap.has(key):
    // MERGE: Same file found by both
    existing = fusionMap.get(key)

    // Merge line ranges
    existingRange = parseRange(existing.location.file)
    newRange = parseRange(semanticResult.location.file)
    mergedRange = {
      start: min(existingRange.start, newRange.start),
      end: max(existingRange.end, newRange.end)
    }

    // Merge content
    mergedContent = existing.content + '\n...\n' + semanticResult.content

    // Boost score
    boostedScore = min(max(existing.scores.lexicalScore,
                           semanticResult.scores.semanticScore) * 1.1, 1.0)

    existing.content = mergedContent
    existing.location.file = updateRange(mergedRange)
    existing.scores = {
      type: 'hybrid',
      lexicalScore: existing.scores.lexicalScore,
      semanticScore: semanticResult.scores.semanticScore,
      compound: boostedScore
    }
  else:
    // New result (semantic only)
    rrfScore = 1 / (60 + semanticRank)
    fusionMap.set(key, { ...semanticResult, scores: { compound: rrfScore } })

// 3. Sort by compound score and return top-N
return Array.from(fusionMap.values())
  .sort((a, b) => b.scores.compound - a.scores.compound)
  .slice(0, maxResults)
```

### 5.5 Merge Example

**Scenario:** Query "AMM pricing"

| Search | Rank | File | Range | Content Snippet |
|--------|------|------|-------|-----------------|
| Lexical | 3 | `docs/AIMM/Pricing.md` | :10-25 | "The AMM pricing mechanism uses..." |
| Semantic | 5 | `docs/AIMM/Pricing.md` | :20-35 | "...pricing is determined by..." |

**After Merge:**

| Origin | File | Range | Content | Score |
|--------|------|-------|---------|-------|
| `hybrid` 🔗 | `docs/AIMM/Pricing.md` | :10-35 | "The AMM pricing mechanism uses...\\n...\\n...pricing is determined by..." | `max(lexical, semantic) × 1.1` |

---

## 6. Scoring & Ranking

### 6.1 Score Components

| Component | Type | Range | Source | Used In |
|-----------|------|-------|--------|--------|
| `lexicalScore` | float | 0-1 | BM25 | Hybrid boost calculation |
| `semanticScore` | float | 0-1 | Cosine similarity | Hybrid boost calculation |
| `lexicalRank` | int | 1-N | BM25 position | RRF + metadata |
| `semanticRank` | int | 1-N | Semantic position | RRF + metadata |
| `lexicalMatchCount` | int | 0-N | Term matches | Metadata only |
| `compound` | float | 0-1 | **Final score** | Ranking |

### 6.2 Final Ranking

After fusion, results are sorted by `compound` score descending:

```typescript
results.sort((a, b) => b.scores.compound - a.scores.compound)
```

### 6.3 Score Flow Diagram

```
┌─────────────────┐
│  Lexical Search  │ → BM25 score (raw) → normalize to 0-1 → lexicalScore
└─────────────────┘

┌─────────────────┐
│ Semantic Search  │ → Cosine similarity → semanticScore
└─────────────────┘

         │
         ▼
┌─────────────────┐
│   RRF Fusion     │ → 1/(60+rank) for each → sum (if hybrid: ×1.1)
└─────────────────┘
         │
         ▼
┌─────────────────┐
│  compound       │ → Used for final ranking
└─────────────────┘
```

---

## 7. Context Formatting

### 7.1 Prioritization

Before formatting, results are re-sorted by origin priority:

```typescript
originPriority = { hybrid: 3, semantic: 2, lexical: 1 }

// This ensures hybrid results appear first in context,
// even if a semantic-only result has a slightly higher compound score
results.sort((a, b) => {
  const priorityDiff = originPriority[b.origin] - originPriority[a.origin]
  if (priorityDiff !== 0) return priorityDiff
  return b.scores.compound - a.scores.compound
})
```

### 7.2 Markdown Format

```markdown
### [Solidity contract] contracts/src/Pool.sol:125-145 🔗

```solidity
function addLiquidity(uint256 amount) external {
    _mint(msg.sender, amount);
    _updateReserves();
}
```

### [TypeScript source] sdk/src/pool.ts:50-65 📝

```typescript
export class Pool {
    async addLiquidity(amount: bigint) {
        // Implementation
    }
}
```
```

### 7.3 Icons

| Origin | Icon | Meaning |
|--------|------|---------|
| 🔗 | Hybrid | Found by both lexical AND semantic |
| 🧠 | Semantic | Found by vector similarity |
| 📝 | Lexical | Found by BM25 keyword matching |

### 7.4 Truncation Rules

| Config | Default | Description |
|--------|---------|-------------|
| `maxResults` | 5 | Max results sent to LLM |
| `maxCharsPerResult` | 500 | Per-result truncation |
| `maxTotalChars` | 4000 | Total context budget |

If truncation occurs, content ends with `...` to indicate omission.

---

## 8. Performance Characteristics

### 8.1 Timing Breakdown

| Stage | Typical Duration | Notes |
|-------|-----------------|-------|
| Query optimization | 200-500ms | LLM call, can be skipped |
| Lexical search | 10-50ms | In-memory BM25 index |
| Semantic search | 100-300ms | TEI embedding + LanceDB |
| RRF fusion | <1ms | In-memory map operations |
| **Total (hybrid)** | **300-800ms** | Parallel lexical + semantic |

### 8.2 Scalability

| Metric | Value |
|--------|-------|
| Max concurrent embeddings | 4 |
| Embedding batch size | 32 |
| Index size (typical) | 10-50K chunks |
| Search latency (p99) | <1s |

---

## 9. Configuration Reference

### 9.1 Search Configuration

```typescript
// back/agents/search/config.ts
export const contextConfig = {
  maxTotalChars: 4000,      // Total context budget
  maxResults: 5,            // Results passed to LLM
  maxCharsPerResult: 500,   // Per-result truncation
};

export const embeddingConfig = {
  model: 'onnx-community/embeddinggemma-300m-ONNX',
  dimensions: 768,
  teiUrl: 'http://localhost:8080/embed',
  batchSize: 32,
  maxConcurrency: 4
};

export const embeddingChunkingConfig = {
  chunkSize: 840,
  overlap: 160,
  minChunkSize: 100
};
```

### 9.2 Agent Configuration

```typescript
// AgentConfig.retrieval
{
  maxResults: 10;            // Top-k from each search method
  enableRAG: true;           // Enable semantic search
  useHybridSearch: true;     // Use RRF fusion
}
```

### 9.3 BM25 Parameters

```typescript
const BM25_PARAMS = {
  k1: 1.2,   // Term frequency saturation
  b: 0.75,   // Length normalization
};

const FUZZY_PARAMS = {
  enabled: true,
  maxDistance: 0.2,     // Max normalized edit distance
  minLength: 4,         // Only apply fuzzy to words >= 4 chars
};
```

### 9.4 RRF Parameters

```typescript
const RRF_K = 60;              // Damping constant
const HYBRID_BOOST = 1.1;      // 10% boost for hybrid results
const HYBRID_BOOST_CAP = 1.0;  // Maximum score after boost
```

---

## Appendix A: File Locations

| Component | Path |
|-----------|------|
| Main hybrid search | `back/agents/search/hybrid.ts` |
| Lexical search | `back/agents/search/lexical/search.ts` |
| Semantic search | `back/agents/search/semantic/search.ts` |
| Query optimizer | `back/agents/search/optimizer.ts` |
| Response handlers | `back/agents/handlers.ts` |
| Vector database | `back/agents/search/vector.ts` |
| Indexer | `back/agents/search/indexer.ts` |
| Configuration | `back/agents/search/config.ts` |
| Type definitions | `back/types.ts` |

---

## Appendix B: Testing

Run hybrid search tests:

```bash
# 1. Start TEI (if not running)
docker run --name tei-embeddings -p 8080:80 \
  ghcr.io/huggingface/text-embeddings-inference:cpu-latest \
  --model-id onnx-community/embeddinggemma-300m-ONNX

# 2. Index knowledge
bun run back/agents/search/index-knowledge.ts

# 3. Run tests
bun run back/agents/search/test-hybrid.ts
bun run back/agents/search/test-hybrid-detailed.ts
```
