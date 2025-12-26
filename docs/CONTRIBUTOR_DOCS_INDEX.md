# Tempo Documentation Index for Contributors

This document provides a guide to all documentation available for contributing to Tempo.

## Getting Started as a Contributor

Start here if you're new to the project:

1. **[ONBOARDING.md](../ONBOARDING.md)** - Comprehensive onboarding guide
   - What is Tempo and why it exists
   - Architecture overview
   - Repository structure
   - Development workflow
   - Testing strategy
   - Common tasks and commands

2. **[QUICKSTART.md](./sources/tempo/QUICKSTART.md)** - Quick reference guide
   - 5-minute setup
   - Essential commands
   - Common development tasks
   - API endpoints
   - TraceQL examples

3. **[CODE_FLOW.md](./CODE_FLOW.md)** - Understanding the codebase
   - Trace ingestion flow
   - Trace query flow
   - Background operations
   - Key code paths to explore

## Contributing

- **[CONTRIBUTING.md](../CONTRIBUTING.md)** - How to contribute
  - Dependency management
  - Project structure
  - Coding standards
  - Testing requirements
  - Linting and formatting

- **[WORKFLOW.md](../WORKFLOW.md)** - Git and release workflow
  - Proposing changes
  - PR requirements
  - Branch structure
  - Release process

- **[GOVERNANCE.md](../GOVERNANCE.md)** - Project governance
  - Team structure
  - Decision making process

## Architecture & Design

- **[Architecture Documentation](./sources/tempo/operations/architecture.md)** - System architecture
  - Component descriptions
  - Operational implications

- **[Design Proposals](./design-proposals/)** - Major design decisions
  - Parquet format
  - TraceQL design
  - Metrics generator
  - vParquet3 format

## Project Structure

Key directories to understand:

```
tempo/
├── cmd/                    # Executables (tempo, tempo-cli, etc.)
├── modules/                # Core components (distributor, ingester, etc.)
├── tempodb/                # Storage layer
├── pkg/                    # Shared packages
├── integration/            # E2E tests
├── example/                # Example deployments
├── docs/                   # Documentation (you are here)
└── operations/             # Deployment resources
```

## Development Resources

### Building and Testing

See [QUICKSTART.md](./sources/tempo/QUICKSTART.md) for quick commands, or:

- Build: `make tempo`
- Test: `make test`
- Lint: `make lint`
- Format: `make fmt`

### Examples

The `example/docker-compose/` directory contains various deployment scenarios:

- **local/** - Simple local deployment
- **distributed/** - Microservices deployment
- **scalable-single-binary/** - Scalable single binary
- **debug/** - Setup for debugging
- And more...

### Tools

- **tempo-cli** - CLI for inspecting blocks and querying traces
- **tempo-vulture** - Consistency checking tool
- **tempo-query** - Jaeger query plugin

## Community Resources

- **Slack**: [#tempo channel](https://grafana.slack.com/archives/C01D981PEE5)
- **Forum**: https://community.grafana.com/c/grafana-tempo/40
- **Issues**: https://github.com/grafana/tempo/issues

## User Documentation

The main user-facing documentation is at:
https://grafana.com/docs/tempo/latest/

Key user docs sections:
- Getting started
- Configuration
- TraceQL
- Operations
- Troubleshooting

## Documentation for Specific Topics

### TraceQL
- [Architecture](./sources/tempo/traceql/architecture.md)
- Design proposals in `design-proposals/2023-11 TraceQL*.md`

### Metrics Generator
- [Overview](./sources/tempo/metrics-generator/)
- Design proposal: `design-proposals/2022-01 Metrics-generator.md`

### Storage Format
- Current: vParquet3 - `design-proposals/2023-05 vParquet3.md`
- Previous: Parquet - `design-proposals/2022-04 Parquet.md`

## Maintaining Documentation

### Writing Documentation

See the [Contributing guide](../CONTRIBUTING.md#documentation) for:
- Documentation structure
- Writing guidelines (Writer's Toolkit)
- Preview locally with `make docs`
- Publishing process

### Documentation Directories

- `docs/sources/tempo/` - Published product documentation
- `docs/design-proposals/` - Internal design documents (not published)
- `docs/internal/` - Internal process documentation

## Quick Links by Role

### New Contributors
1. [ONBOARDING.md](../ONBOARDING.md)
2. [QUICKSTART.md](./sources/tempo/QUICKSTART.md)
3. Pick a `good first issue` on GitHub
4. Join Slack

### Code Contributors
1. [CODE_FLOW.md](./CODE_FLOW.md)
2. [CONTRIBUTING.md](../CONTRIBUTING.md)
3. [Architecture docs](./sources/tempo/operations/architecture.md)
4. Relevant design proposals

### Documentation Contributors
1. [Writer's Toolkit](https://grafana.com/docs/writers-toolkit/)
2. [Contributing - Documentation](../CONTRIBUTING.md#documentation)
3. `docs/sources/tempo/` for product docs
4. Use `type/doc` label on PRs

### Reviewers
1. [WORKFLOW.md](../WORKFLOW.md)
2. [GOVERNANCE.md](../GOVERNANCE.md)
3. [Architecture docs](./sources/tempo/operations/architecture.md)

## Finding Your Way Around

**Lost?** Here's what to read based on what you want to do:

| I want to... | Read this... |
|-------------|--------------|
| Understand what Tempo is | [ONBOARDING.md](../ONBOARDING.md) - "What is Tempo?" |
| Set up my dev environment | [QUICKSTART.md](./sources/tempo/QUICKSTART.md) - "5-Minute Setup" |
| Understand the architecture | [Architecture docs](./sources/tempo/operations/architecture.md) |
| Trace a request through code | [CODE_FLOW.md](./CODE_FLOW.md) |
| Run tests | [QUICKSTART.md](./sources/tempo/QUICKSTART.md) - "Testing" |
| Add a feature | [CONTRIBUTING.md](../CONTRIBUTING.md) + relevant design docs |
| Fix a bug | [CODE_FLOW.md](./CODE_FLOW.md) + [QUICKSTART.md](./sources/tempo/QUICKSTART.md) |
| Deploy Tempo | `example/` directory + user docs |
| Write documentation | [CONTRIBUTING.md](../CONTRIBUTING.md#documentation) |
| Propose a major change | [WORKFLOW.md](../WORKFLOW.md) - "Proposing changes" |

## Additional Resources

- **Code of Conduct**: [CODE_OF_CONDUCT.md](../CODE_OF_CONDUCT.md)
- **Security**: [SECURITY.md](../SECURITY.md)
- **License**: [LICENSE](../LICENSE) (AGPL-3.0) and [LICENSING.md](../LICENSING.md)
- **Adopters**: [ADOPTERS.md](../ADOPTERS.md)
- **Maintainers**: [MAINTAINERS.md](../MAINTAINERS.md)
- **Changelog**: [CHANGELOG.md](../CHANGELOG.md)
- **Releases**: [RELEASES.MD](../RELEASES.MD)

---

**Questions?** Ask in [#tempo on Slack](https://grafana.slack.com/archives/C01D981PEE5) or [open an issue](https://github.com/grafana/tempo/issues/new/choose)!
