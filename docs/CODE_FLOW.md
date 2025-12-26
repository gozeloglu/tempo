# Understanding Tempo Code Flow

This document explains how traces flow through the Tempo codebase, helping you understand where to look for different functionality.

## Trace Ingestion Flow

Here's how a trace gets ingested and stored in Tempo:

```
┌─────────────┐
│   Client    │ (Application sending traces)
│  (OTel SDK) │
└──────┬──────┘
       │ HTTP/gRPC (OTLP, Jaeger, Zipkin)
       ▼
┌─────────────────────────────────────────────────────────┐
│ DISTRIBUTOR (modules/distributor/)                      │
│                                                          │
│  1. Receives spans via multiple protocols               │
│     • OpenTelemetry: receiver.go, otlp.go              │
│     • Jaeger: receiver.go                               │
│     • Zipkin: receiver.go                               │
│                                                          │
│  2. Validates spans                                     │
│     • Size limits                                       │
│     • Required fields                                   │
│                                                          │
│  3. Shards by trace ID                                  │
│     • Hash(traceID) % num_ingesters                     │
│     • Uses consistent hashing (dskit)                   │
│                                                          │
│  Key files: distributor.go, forwarder.go               │
└───────────────────────┬─────────────────────────────────┘
                        │ Push spans
                        ▼
┌─────────────────────────────────────────────────────────┐
│ INGESTER (modules/ingester/)                           │
│                                                          │
│  1. Receives spans for specific trace IDs              │
│                                                          │
│  2. Buffers in memory (WAL)                            │
│     • Write-Ahead Log: tempodb/wal/                    │
│     • In-memory trace assembly                          │
│                                                          │
│  3. Creates blocks periodically                         │
│     • Batches traces together                          │
│     • Generates bloom filters                          │
│     • Creates indexes                                   │
│                                                          │
│  4. Flushes to backend storage                         │
│     • Uploads to S3/GCS/Azure                          │
│     • Block format: tempodb/encoding/                  │
│                                                          │
│  Key files: ingester.go, instance.go, flush.go         │
└───────────────────────┬─────────────────────────────────┘
                        │ Flush blocks
                        ▼
┌─────────────────────────────────────────────────────────┐
│ BACKEND STORAGE (tempodb/backend/)                      │
│                                                          │
│  Storage implementations:                               │
│  • S3:    backend/s3/                                  │
│  • GCS:   backend/gcs/                                 │
│  • Azure: backend/azure/                               │
│  • Local: backend/local/                               │
│                                                          │
│  Block structure:                                       │
│  <tenant>/<blockID>/meta.json    (metadata)            │
│                   /data          (trace data)          │
│                   /index         (search index)        │
│                   /bloom_*       (bloom filters)       │
│                                                          │
│  Key files: backend/backend.go, encoding/vparquet3/    │
└─────────────────────────────────────────────────────────┘
```

## Trace Query Flow

Here's how traces are retrieved from Tempo:

```
┌─────────────┐
│  Grafana    │ (or any HTTP client)
│     UI      │
└──────┬──────┘
       │ GET /api/traces/<traceID>
       ▼
┌─────────────────────────────────────────────────────────┐
│ QUERY FRONTEND (modules/frontend/)                      │
│                                                          │
│  1. Receives query request                              │
│     • Trace ID lookup                                   │
│     • TraceQL search query                              │
│                                                          │
│  2. Shards the query                                    │
│     • Splits blockID space                             │
│     • Creates sub-queries                               │
│     • Queues requests                                   │
│                                                          │
│  3. Distributes to queriers                            │
│     • gRPC streaming                                    │
│     • Load balancing                                    │
│                                                          │
│  Key files: frontend.go, query_frontend.go             │
└───────────────────────┬─────────────────────────────────┘
                        │ Sharded queries
                        ▼
┌─────────────────────────────────────────────────────────┐
│ QUERIER (modules/querier/)                             │
│                                                          │
│  1. Receives sharded query                             │
│                                                          │
│  2. Checks ingesters first (recent traces)             │
│     • Queries all ingesters                            │
│     • Gets in-memory traces                            │
│                                                          │
│  3. Searches backend storage                           │
│     • Downloads bloom filters                          │
│     • Checks if trace exists                           │
│     • Downloads block data if found                     │
│                                                          │
│  4. Assembles and returns trace                        │
│     • Combines all spans                               │
│     • Sorts by timestamp                                │
│                                                          │
│  Key files: querier.go, worker.go                      │
└───────────────────────┬─────────────────────────────────┘
                        │ Trace data
                        ▼
┌─────────────────────────────────────────────────────────┐
│ TEMPODB (tempodb/)                                      │
│                                                          │
│  Query operations:                                      │
│  • FindTraceByID(): tempodb.go                         │
│  • SearchTags(): tempodb_search.go                     │
│  • TraceQL queries: backend/block.go                   │
│                                                          │
│  Uses bloom filters for efficiency:                     │
│  • bloom_filter.go                                     │
│  • Reduces unnecessary block reads                      │
│                                                          │
│  Block readers:                                         │
│  • encoding/vparquet3/block_traceql.go                 │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## Background Operations

### Compaction

```
┌─────────────────────────────────────────────────────────┐
│ COMPACTOR (modules/compactor/)                          │
│                                                          │
│  Runs periodically to:                                  │
│  1. List blocks in backend storage                      │
│  2. Select blocks for compaction                        │
│     • Based on size, age, level                         │
│     • compaction_block_selector.go                      │
│  3. Merge blocks                                        │
│     • Combines multiple blocks                          │
│     • Rebuilds indexes and bloom filters                │
│  4. Upload compacted block                              │
│  5. Mark old blocks for deletion                        │
│                                                          │
│  Key files: compactor.go, compaction.go                │
└─────────────────────────────────────────────────────────┘
```

### Metrics Generation (Optional)

```
┌─────────────────────────────────────────────────────────┐
│ METRICS GENERATOR (modules/generator/)                  │
│                                                          │
│  Derives metrics from traces:                           │
│  1. Receives spans (like distributor)                   │
│  2. Extracts metrics:                                   │
│     • Span duration → latency histogram                │
│     • Span count → request rate                         │
│     • Status → error rate                               │
│  3. Aggregates by dimensions:                           │
│     • Service name                                      │
│     • Operation name                                    │
│     • Custom attributes                                 │
│  4. Writes to metrics backend:                          │
│     • Prometheus remote write                           │
│     • Or local storage                                  │
│                                                          │
│  Key files: generator.go, processor.go                 │
└─────────────────────────────────────────────────────────┘
```

## Key Code Paths to Explore

### 1. Adding Support for a New Span Format

Start here:
- `modules/distributor/receiver.go` - Protocol receivers
- Add your receiver using OTel Collector components
- Wire it up in `cmd/tempo/app/app.go`

### 2. Understanding Block Format

Start here:
- `tempodb/encoding/vparquet3/` - Current block format
- `tempodb/encoding/common/` - Shared encoding logic
- Design doc: `docs/design-proposals/2023-05 vParquet3.md`

### 3. Implementing a TraceQL Feature

Start here:
- `pkg/traceql/` - TraceQL parsing and AST
- `tempodb/encoding/vparquet3/block_traceql.go` - Query execution
- `modules/frontend/tracql_metrics.go` - Metrics queries

### 4. Adding a New Storage Backend

Start here:
- `tempodb/backend/backend.go` - Backend interface
- Look at `tempodb/backend/s3/` as example
- Implement: Reader, Writer, Compactor interfaces

### 5. Modifying Query Behavior

Start here:
- `modules/querier/querier.go` - Main query logic
- `modules/frontend/frontend.go` - Query sharding
- `tempodb/tempodb.go` - Storage query interface

## Important Concepts in Code

### 1. Trace IDs and Sharding

```go
// modules/distributor/distributor.go
func (d *Distributor) Push(ctx context.Context, req *tempopb.PushRequest) {
    // Trace ID determines which ingester receives the span
    key := util.TokenFor(req.TraceID)
    ingester := d.ring.Get(key)
    // ...
}
```

### 2. Bloom Filters

```go
// tempodb/bloom/bloom.go
// Probabilistic data structure for fast negative lookups
type Bloom struct {
    // If Test() returns false, trace definitely NOT in block
    // If Test() returns true, trace MIGHT be in block
}
```

### 3. Block Metadata

```go
// tempodb/backend/block.go
type BlockMeta struct {
    BlockID      uuid.UUID
    CompactionLevel int
    TotalObjects int
    StartTime    time.Time
    EndTime      time.Time
    // ... more fields
}
```

### 4. WAL (Write-Ahead Log)

```go
// tempodb/wal/wal.go
// Provides durability before blocks are flushed
type WAL struct {
    // Writes spans to disk immediately
    // Allows recovery after crash
}
```

## Testing Your Changes

### Unit Tests

```bash
# Test specific component
go test ./modules/distributor/... -v
go test ./tempodb/encoding/... -v
```

### Integration Tests

```bash
# E2E tests with real storage
make test-e2e

# Specific scenario
cd integration/e2e
go test -run TestSearchTraceQL -v
```

### Local Testing

```bash
# Modify code, rebuild, and test
cd example/docker-compose/local
docker compose build tempo
docker compose up tempo

# Send test traces
# Use your application or k6-tracing container
```

## Debugging Tips

### 1. Enable Debug Logging

In your tempo config:
```yaml
server:
  log_level: debug
```

### 2. Use tempo-cli

```bash
# Inspect a block
tempo-cli analyse blocks --backend=local --bucket=./tempo-data

# Query directly
tempo-cli query api traces <trace-id> http://localhost:3200
```

### 3. Add Logging

```go
import "github.com/go-kit/log/level"

level.Debug(logger).Log("msg", "my debug message", "traceID", traceID)
```

### 4. Use Delve Debugger

```bash
cd example/docker-compose/debug
# Follow README to attach debugger
```

## Common Pitfalls

1. **Forgetting to vendor**: Run `make vendor-check` after changing dependencies
2. **Wrong import order**: Use `make fmt` to fix
3. **Not testing with bloom filters**: Test both with and without filters
4. **Ignoring block compaction**: Consider compacted blocks in your logic
5. **Not handling multi-tenancy**: Use orgID/tenantID correctly

## Further Reading

- **Protocol buffers**: `pkg/tempopb/*.proto`
- **Configuration**: See `modules/*/config.go` files
- **Design proposals**: `docs/design-proposals/*.md`
- **Monitoring**: `operations/tempo-mixin/`

---

**Questions?** Ask in the [Tempo Slack channel](https://grafana.slack.com/archives/C01D981PEE5)
