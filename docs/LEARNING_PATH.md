# Learning Path: Understanding Tempo

This guide provides a structured learning path for understanding the Tempo codebase. Follow this path to build your knowledge progressively.

## Overview

This learning path is designed for:
- New contributors who want to understand the codebase
- Engineers evaluating Tempo for their organization
- Anyone curious about how distributed tracing works at scale

**Estimated time**: 2-4 weeks (depending on your pace and Go experience)

## Prerequisites

- Basic understanding of distributed systems
- Familiarity with Go programming language
- Understanding of HTTP/gRPC basics
- Basic knowledge of observability concepts (logs, metrics, traces)

If you're new to any of these, don't worry! You can learn as you go.

## Learning Path

### Week 1: Foundation

#### Day 1-2: What and Why

**Goal**: Understand what Tempo is and the problems it solves

1. **Read the basics**:
   - [README.md](../README.md) - Project overview
   - [Introduction docs](./sources/tempo/introduction/_index.md) - What are traces?
   - [Architecture](./sources/tempo/operations/architecture.md) - High-level architecture

2. **Watch videos**:
   - [How to get started with Tempo](https://www.youtube.com/watch?v=zDrA7Ly3ovU)
   - [What traces provide that logs and metrics don't](https://www.youtube.com/watch?v=0tlp7QCPu0k)

3. **Explore traces**:
   - Run the local example: `example/docker-compose/local`
   - Send some traces
   - Query them in Grafana
   - Observe the data flow

**Checkpoint**: Can you explain what a trace is and why Tempo exists?

#### Day 3-4: Getting Hands-On

**Goal**: Set up development environment and build the project

1. **Setup**:
   - Follow [QUICKSTART.md](./sources/tempo/QUICKSTART.md) - "5-Minute Setup"
   - Build Tempo: `make tempo`
   - Run tests: `make test` (just to see they work)

2. **Explore the repository**:
   - Walk through [ONBOARDING.md](../ONBOARDING.md) - "Repository Structure"
   - Navigate the key directories
   - Open and skim key files mentioned

3. **Run examples**:
   - Try `example/docker-compose/local`
   - Try `example/docker-compose/distributed`
   - Compare the two setups

**Checkpoint**: Can you build Tempo and run a local instance?

#### Day 5-7: Understanding Data Flow

**Goal**: Understand how data flows through Tempo

1. **Read the flow docs**:
   - [CODE_FLOW.md](./CODE_FLOW.md) - Complete data flow
   - Focus on "Trace Ingestion Flow"

2. **Trace the code**:
   - Start at `cmd/tempo/main.go`
   - Follow initialization in `cmd/tempo/app/app.go`
   - Find where distributor is created
   - Find where ingester is created

3. **Read key files**:
   - `modules/distributor/distributor.go` - Entry point for spans
   - `modules/ingester/ingester.go` - Batching and storage
   - `tempodb/tempodb.go` - Storage interface

**Checkpoint**: Can you trace a span from API call to storage?

### Week 2: Deep Dive into Components

#### Day 8-10: Distributor and Ingester

**Goal**: Understand ingestion path in detail

1. **Distributor deep dive**:
   - Read all files in `modules/distributor/`
   - Understand receiver layer
   - Understand how trace IDs are hashed
   - Look at validation logic

2. **Ingester deep dive**:
   - Read all files in `modules/ingester/`
   - Understand instances and traces
   - Understand WAL (`tempodb/wal/`)
   - Understand flushing logic

3. **Hands-on**:
   - Add debug logging to distributor
   - Rebuild and run local example
   - Send traces and observe logs
   - See where your traces go

**Exercise**: Modify the distributor to log every trace ID it receives.

**Checkpoint**: Can you explain how spans become blocks?

#### Day 11-13: Querier and Query Frontend

**Goal**: Understand query path in detail

1. **Query Frontend**:
   - Read `modules/frontend/frontend.go`
   - Understand query sharding
   - Understand how work is distributed

2. **Querier deep dive**:
   - Read `modules/querier/querier.go`
   - Understand how it queries ingesters
   - Understand how it queries backend storage

3. **Read query flow**:
   - [CODE_FLOW.md](./CODE_FLOW.md) - "Trace Query Flow"
   - Follow a query from API to response

4. **Hands-on**:
   - Query traces using the API
   - Query traces using tempo-cli
   - Look at the logs while querying

**Exercise**: Use tempo-cli to query a trace and understand each log line.

**Checkpoint**: Can you explain how a trace is retrieved?

#### Day 14: Storage Layer (TempoDB)

**Goal**: Understand how Tempo stores data

1. **TempoDB overview**:
   - Read `tempodb/tempodb.go`
   - Understand the Reader/Writer interfaces
   - Look at block metadata

2. **Block format**:
   - Read `tempodb/encoding/common/types.go`
   - Look at `tempodb/encoding/vparquet3/`
   - Read design proposal: `design-proposals/2023-05 vParquet3.md`

3. **Backend storage**:
   - Look at `tempodb/backend/backend.go`
   - Examine one backend (e.g., `tempodb/backend/local/`)
   - Understand block structure on disk

4. **Bloom filters**:
   - Read about bloom filters in `tempodb/bloom/`
   - Understand why they're used

**Exercise**: Use tempo-cli to inspect a block in your local storage.

**Checkpoint**: Can you explain the block format and why bloom filters are used?

### Week 3: Advanced Topics

#### Day 15-17: TraceQL

**Goal**: Understand Tempo's query language

1. **TraceQL basics**:
   - Read TraceQL docs: `docs/sources/tempo/traceql/`
   - Try queries in Grafana UI
   - Read [Architecture](./sources/tempo/traceql/architecture.md)

2. **TraceQL implementation**:
   - Look at `pkg/traceql/` - Parser and AST
   - Look at `tempodb/encoding/vparquet3/block_traceql.go` - Query execution
   - Read design proposal: `design-proposals/2022-04 TraceQL Concepts.md`

3. **Hands-on**:
   - Run various TraceQL queries
   - Look at how they're executed
   - Try to understand query planning

**Exercise**: Write a complex TraceQL query and trace its execution.

**Checkpoint**: Can you write TraceQL queries and understand how they execute?

#### Day 18-19: Compaction

**Goal**: Understand background processes

1. **Compactor**:
   - Read `modules/compactor/compactor.go`
   - Understand why compaction is needed
   - Look at `tempodb/compaction_block_selector.go`

2. **Hands-on**:
   - Run Tempo long enough to see compaction
   - Use tempo-cli to see blocks before/after compaction
   - Observe block levels changing

**Checkpoint**: Can you explain why and how blocks are compacted?

#### Day 20-21: Optional Components

**Goal**: Understand metrics generator and other optional features

1. **Metrics Generator**:
   - Read `modules/generator/`
   - Read design proposal: `design-proposals/2022-01 Metrics-generator.md`
   - Try the metrics generator example

2. **Other features**:
   - Multi-tenancy: Look at `modules/overrides/`
   - Caching: Look at `modules/cache/`
   - Block builder: Look at `modules/blockbuilder/`

**Checkpoint**: Can you explain what the metrics generator does?

### Week 4: Contributing

#### Day 22-24: Making Changes

**Goal**: Make your first contribution

1. **Find an issue**:
   - Look for `good first issue` labels
   - Or find something that confused you to document

2. **Make a change**:
   - Follow [CONTRIBUTING.md](../CONTRIBUTING.md)
   - Write code or documentation
   - Add tests
   - Submit a PR

3. **Code review**:
   - Respond to feedback
   - Learn from reviewers
   - Iterate

**Checkpoint**: Have you submitted your first PR?

#### Day 25-28: Deep Specialization

**Goal**: Become expert in one area

Choose one area to specialize in:

1. **TraceQL**:
   - Read all TraceQL code
   - Try to add a new function
   - Optimize query performance

2. **Storage**:
   - Deep dive into Parquet format
   - Understand columnar storage
   - Look at performance optimizations

3. **Query performance**:
   - Profile queries
   - Look for bottlenecks
   - Implement caching improvements

4. **Protocol support**:
   - Add support for a new trace format
   - Improve existing receivers

5. **Operations**:
   - Look at monitoring dashboards
   - Understand operational best practices
   - Improve observability

**Checkpoint**: Can you claim expertise in one area?

## Learning Resources by Topic

### Distributed Tracing Concepts
- OpenTelemetry documentation
- Jaeger documentation
- "Distributed Tracing in Practice" book

### Go Programming
- "The Go Programming Language" book
- Effective Go guide
- Go by Example

### Parquet Format
- Apache Parquet documentation
- Parquet design docs in Tempo
- Columnar storage papers

### gRPC and Protocol Buffers
- gRPC documentation
- Protocol Buffers guide
- OpenTelemetry protocol specs

## Practice Exercises

### Exercise 1: Add Logging
Add debug logging throughout the ingestion path. Observe a trace from API to storage.

### Exercise 2: Custom Receiver
Create a custom receiver that accepts traces in a made-up format and converts them to OTLP.

### Exercise 3: Query Optimization
Profile a query and identify bottlenecks. Try to optimize one bottleneck.

### Exercise 4: Storage Backend
Implement a simple storage backend (e.g., using SQLite or a key-value store).

### Exercise 5: TraceQL Function
Add a new function to TraceQL (e.g., `uppercase(str)` or `extract_domain(url)`).

### Exercise 6: Monitoring Dashboard
Create a new Grafana dashboard for a specific Tempo component.

## Assessment: How Well Do You Know Tempo?

Answer these questions to assess your understanding:

1. **Basic**:
   - What is a trace? What is a span?
   - What are Tempo's main components?
   - What storage backends does Tempo support?

2. **Intermediate**:
   - How does Tempo shard spans across ingesters?
   - What is a bloom filter and why does Tempo use them?
   - How does TraceQL query execution work?

3. **Advanced**:
   - Why does Tempo use Parquet format?
   - How does compaction work and why is it needed?
   - What are the trade-offs of different retention strategies?

4. **Expert**:
   - How would you optimize query performance for large traces?
   - How would you design a new storage format?
   - How would you add multi-datacenter support?

## Next Steps

Once you've completed this learning path:

1. **Contribute regularly**: Pick an area you enjoy and keep contributing
2. **Help others**: Answer questions in Slack and GitHub issues
3. **Write blog posts**: Share what you learned
4. **Present at meetups**: Talk about Tempo features you worked on
5. **Become a maintainer**: With sustained contributions, you can become a maintainer

## Getting Help

Stuck on something? That's normal! Here's how to get help:

1. **Search docs**: Check if it's already documented
2. **Read code**: Often code is the best documentation
3. **Ask in Slack**: [#tempo channel](https://grafana.slack.com/archives/C01D981PEE5)
4. **Open an issue**: If something is unclear, it's probably a doc bug
5. **Tag maintainers**: On GitHub, you can tag maintainers for specific areas

## Tips for Success

- **Don't rush**: Understanding takes time
- **Get hands-on**: Reading isn't enough, you need to experiment
- **Ask questions**: No question is too simple
- **Document as you learn**: Help future learners
- **Contribute early**: Even fixing typos helps you learn the process
- **Be patient**: Becoming an expert takes months, not days

---

**Ready to start?** Begin with Week 1, Day 1 and enjoy the journey!

**Questions?** Ask in [#tempo on Slack](https://grafana.slack.com/archives/C01D981PEE5)
