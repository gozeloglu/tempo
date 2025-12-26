# Onboarding Guide: Grafana Tempo

Welcome to Grafana Tempo! This guide will help you understand the codebase and get started with contributing to the project.

## Table of Contents

1. [What is Tempo?](#what-is-tempo)
2. [Architecture Overview](#architecture-overview)
3. [Repository Structure](#repository-structure)
4. [Getting Started](#getting-started)
5. [Development Workflow](#development-workflow)
6. [Testing Strategy](#testing-strategy)
7. [Common Tasks](#common-tasks)
8. [Key Concepts](#key-concepts)
9. [Resources](#resources)

## What is Tempo?

Grafana Tempo is an open-source, easy-to-use, and high-scale distributed tracing backend. Key features:

- **Cost-efficient**: Requires only object storage (S3, GCS, Azure) to operate
- **Protocol support**: Compatible with Jaeger, Zipkin, Kafka, OpenCensus, and OpenTelemetry
- **TraceQL**: A powerful traces-first query language for targeted queries
- **Deep integration**: Works seamlessly with Grafana, Prometheus, and Loki

### What Problem Does Tempo Solve?

Distributed tracing helps teams:
- Quickly pinpoint performance issues
- Understand request flow across microservices
- Identify bottlenecks and latency issues
- Debug complex distributed systems

## Architecture Overview

Tempo uses a microservices architecture with the following main components:

```
┌─────────────┐     ┌──────────────┐     ┌────────────┐
│ Distributor │────▶│   Ingester   │────▶│  Backend   │
└─────────────┘     └──────────────┘     │  Storage   │
       │                                  │ (S3/GCS/   │
       │                                  │  Azure)    │
       ▼                                  └────────────┘
┌─────────────┐     ┌──────────────┐           │
│Query Frontend│────▶│   Querier    │───────────┘
└─────────────┘     └──────────────┘
                           │
                    ┌──────────────┐
                    │  Compactor   │
                    └──────────────┘
```

### Component Descriptions

1. **Distributor**: 
   - Accepts spans in multiple formats (Jaeger, OpenTelemetry, Zipkin)
   - Routes spans to ingesters using consistent hashing based on traceID
   - Uses OpenTelemetry Collector receiver layer

2. **Ingester**:
   - Batches traces into blocks
   - Creates bloom filters and indexes
   - Flushes data to backend storage

3. **Query Frontend**:
   - Shards search space for incoming queries
   - Exposes HTTP endpoint: `GET /api/traces/<traceID>`
   - Distributes work to queriers

4. **Querier**:
   - Finds traces in ingesters (recent) or backend storage
   - Uses bloom filters for efficient trace lookup
   - Should not be accessed directly (use Query Frontend)

5. **Compactor**:
   - Reduces total number of blocks in backend storage
   - Streams blocks to/from storage for compaction

6. **Metrics Generator** (optional):
   - Derives metrics from ingested traces
   - Writes metrics to a metrics storage backend

## Repository Structure

```
tempo/
├── cmd/                      # Main binaries
│   ├── tempo/               # Main Tempo binary
│   ├── tempo-cli/          # CLI tool for inspecting blocks
│   ├── tempo-query/        # Jaeger-query GRPC plugin
│   └── tempo-vulture/      # Consistency checker tool
├── modules/                 # Top-level Tempo components
│   ├── distributor/        # Distributor implementation
│   ├── ingester/           # Ingester implementation
│   ├── querier/            # Querier implementation
│   ├── frontend/           # Query frontend implementation
│   ├── compactor/          # Compactor implementation
│   ├── generator/          # Metrics generator
│   ├── overrides/          # Tenant limits/overrides
│   ├── storage/            # Storage interfaces
│   ├── cache/              # Caching layer
│   └── blockbuilder/       # Block building logic
├── tempodb/                # Object storage key/value database
│   ├── backend/            # Storage backend implementations
│   ├── encoding/           # Block encoding formats
│   └── wal/                # Write-ahead log
├── pkg/                    # Shared packages
│   └── tempopb/           # Protobuf definitions
├── integration/            # End-to-end tests
│   └── e2e/               # E2E test suite
├── example/                # Example deployments
│   ├── docker-compose/    # Docker Compose examples
│   ├── helm/              # Helm chart examples
│   └── tk/                # Jsonnet/Tanka examples
├── docs/                   # Documentation
│   ├── sources/tempo/     # Product documentation
│   └── design-proposals/  # Design documents
├── operations/             # Deployment resources
│   ├── jsonnet/           # Jsonnet configs
│   └── tempo-mixin/       # Monitoring mixin
└── vendor/                 # Vendored dependencies
```

## Getting Started

### Prerequisites

- **Go**: Version 1.24.1 or higher (check `go.mod`)
- **Git**: For version control
- **Docker**: For running examples and tests
- **Make**: Build automation

### Initial Setup

1. **Clone the repository**:
   ```bash
   git clone https://github.com/grafana/tempo.git
   cd tempo
   ```

2. **Verify dependencies**:
   ```bash
   make vendor-check
   ```

3. **Build the main binary**:
   ```bash
   make tempo
   # Binary will be in: ./bin/<os>/tempo-<arch>
   ```

4. **Run a quick example**:
   ```bash
   cd example/docker-compose/local
   docker-compose up
   ```
   This starts a local Tempo instance with Grafana for visualization.

### Running Your First Trace

After starting the docker-compose example:

1. Access Grafana at http://localhost:3000 (admin/admin)
2. Send a test trace using the provided tools
3. Query the trace using TraceQL or the trace ID

## Development Workflow

### Code Standards

1. **Import Organization**:
   ```go
   import (
       // Standard library
       "context"
       "fmt"
       
       // External libraries
       "github.com/gogo/protobuf/proto"
       "github.com/opentracing/opentracing-go"
       
       // Local packages
       "github.com/grafana/tempo/modules/overrides"
       "github.com/grafana/tempo/pkg/validation"
   )
   ```

2. **Formatting**:
   ```bash
   make fmt        # Format code with gofumpt and goimports
   make lint       # Run linter
   ```

3. **Dependencies**:
   ```bash
   go get example.com/some/module/pkg@vX.Y.Z  # Add dependency
   make vendor-check                           # Verify consistency
   ```

### Building Components

```bash
# Build all main binaries
make tempo           # Main Tempo binary
make tempo-cli       # CLI tool
make tempo-query     # Query plugin
make tempo-vulture   # Consistency checker

# Build Docker images
make docker-tempo          # Tempo Docker image
make docker-tempo-query    # Query Docker image
```

### Making Changes

1. **Create a branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes** following the coding standards

3. **Format and lint**:
   ```bash
   make fmt
   make lint
   ```

4. **Test your changes** (see Testing Strategy below)

5. **Create a pull request** following the guidelines in CONTRIBUTING.md

## Testing Strategy

Tempo has multiple levels of testing:

### Unit Tests

Located within each package as `*_test.go` files.

```bash
# Run all tests
make test

# Run tests with coverage
make test-with-cover

# Run specific package tests
make test-with-cover-pkg         # Tests in pkg/
make test-with-cover-tempodb     # Tests in tempodb/
make test-with-cover-others      # All other tests
```

### Integration Tests

End-to-end tests that test the complete ingest and query path.

```bash
# Run all E2E tests
make test-e2e

# Run specific E2E test suites
make test-e2e-deployments  # Deployment mode tests
make test-e2e-api          # API tests

# Run poller integration tests
make test-integration-poller
```

### Benchmarks

```bash
make benchmark      # Run benchmarks
make test-bench     # Run benchmark integration tests
```

### Local Testing

Use the examples to test your changes:

```bash
cd example/docker-compose/local
docker-compose up

# In another terminal, rebuild and restart
docker-compose build tempo
docker-compose restart tempo
```

### Debugging

For debugging with a debugger (e.g., Delve):
```bash
cd example/docker-compose/debug
# Follow instructions in that directory
```

## Common Tasks

### Adding a New Configuration Option

1. Add the field to the config struct (e.g., in `modules/distributor/config.go`)
2. Set appropriate defaults
3. Update documentation in `docs/sources/tempo/configuration/`
4. Add validation if needed
5. Write tests

### Adding a New Metric

1. Define the metric in the appropriate module
2. Follow the naming convention: `tempo_<component>_<metric_name>`
3. Add the metric to the monitoring mixin: `operations/tempo-mixin/`
4. Update dashboards if relevant

### Working with Protocol Buffers

```bash
# Proto definitions are in opentelemetry-proto (git submodule)
# and pkg/tempopb/

# Regenerate proto files (if you modify .proto files)
make gen-proto
```

### Working with Jsonnet

```bash
# Format jsonnet files
make jsonnetfmt

# Compile jsonnet
make jsonnet
```

### Updating Dependencies

```bash
# Update a specific dependency
go get github.com/example/package@v1.2.3

# Tidy and vendor
go mod tidy
go mod vendor

# Verify everything is consistent
make vendor-check
```

## Key Concepts

### Traces and Spans

- **Trace**: Complete journey of a request through a distributed system
- **Span**: Single unit of work within a trace
- **Trace ID**: Unique identifier that links all spans in a trace

### Block Format

Tempo stores traces in blocks with the following structure:
```
<bucketname>/<tenantID>/<blockID>/meta.json
                                 /index
                                 /data
                                 /bloom_0
                                 /bloom_1
                                 ...
```

### TraceQL

TraceQL is Tempo's query language for searching traces:

```traceql
# Find traces with errors
{ status = error }

# Find slow HTTP requests
{ http.status_code = 200 && duration > 1s }

# Complex queries
{ .service.name = "frontend" } && { .http.method = "POST" }
```

### Storage Backends

Tempo supports multiple backend storage options:
- **S3** (AWS)
- **GCS** (Google Cloud)
- **Azure Blob Storage**
- **Local disk** (for testing)

### Bloom Filters

Tempo uses bloom filters to efficiently search for traces:
- Probabilistic data structure
- Allows fast negative lookups ("this trace is definitely not in this block")
- Reduces need to download full block for searches

## Resources

### Documentation

- **Main Documentation**: https://grafana.com/docs/tempo/latest/
- **Getting Started**: https://grafana.com/docs/tempo/latest/getting-started/
- **TraceQL Guide**: https://grafana.com/docs/tempo/latest/traceql/
- **Operations Guide**: https://grafana.com/docs/tempo/latest/operations/

### Internal Documentation

- **[CONTRIBUTING.md](CONTRIBUTING.md)**: Contribution guidelines
- **[GOVERNANCE.md](GOVERNANCE.md)**: Project governance
- **[WORKFLOW.md](WORKFLOW.md)**: Git workflow and release process
- **[Architecture Doc](docs/sources/tempo/operations/architecture.md)**: Detailed architecture
- **[Design Proposals](docs/design-proposals/)**: Major design decisions

### Community

- **Slack**: [#tempo channel](https://grafana.slack.com/archives/C01D981PEE5)
- **Forum**: https://community.grafana.com/c/grafana-tempo/40
- **Issues**: https://github.com/grafana/tempo/issues

### Blog Posts & Videos

- [How to get started with Tempo (video)](https://www.youtube.com/watch?v=zDrA7Ly3ovU)
- [Grafana blog posts about Tempo](https://grafana.com/tags/tempo/)
- [TraceQL Introduction](https://grafana.com/blog/2023/02/07/get-to-know-traceql-a-powerful-new-query-language-for-distributed-tracing/)

### Example Traces

After running a local example, try these operations:

1. **Send a trace** using OpenTelemetry SDK
2. **Query by trace ID** via the API or Grafana UI
3. **Search with TraceQL** to find specific traces
4. **View metrics** derived from traces (if using metrics generator)

## Next Steps

Now that you're familiar with Tempo, here are some suggested next steps:

1. **Explore the codebase**: Start with `cmd/tempo/main.go` and follow the code flow
2. **Run the examples**: Try different deployment modes in `example/`
3. **Read design proposals**: Understand major architectural decisions in `docs/design-proposals/`
4. **Pick a good first issue**: Look for issues labeled `good first issue` on GitHub
5. **Join the community**: Say hi in Slack and introduce yourself

## Quick Reference

### Makefile Targets

```bash
make help              # Show all available targets
make tempo            # Build main binary
make test             # Run tests
make lint             # Run linter
make fmt              # Format code
make docker-tempo     # Build Docker image
make docs             # Preview documentation locally
```

### Key Directories to Know

- `modules/distributor/`: Span ingestion logic
- `modules/querier/`: Trace query logic
- `tempodb/`: Storage layer implementation
- `pkg/tempopb/`: Protocol buffer definitions
- `integration/e2e/`: End-to-end tests

### Common Commands

```bash
# Build and test cycle
make tempo && make test

# Lint before committing
make fmt && make lint

# Run local example
cd example/docker-compose/local && docker-compose up

# View logs from tests
make test 2>&1 | tee test.log
```

---

**Welcome to the Tempo community! Happy coding! 🎉**

If you have questions or need help, don't hesitate to ask in the Slack channel or on the forum.
