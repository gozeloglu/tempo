---
title: Quick Start Guide
description: Quick reference guide for getting started with Tempo development
weight: 50
---

# Quick Start Guide

This is a quick reference guide for developers who want to get started with Tempo development quickly.

## Prerequisites Checklist

- [ ] Go 1.24.1+ installed
- [ ] Docker installed and running
- [ ] Git configured
- [ ] Make available

## 5-Minute Setup

```bash
# 1. Clone the repository
git clone https://github.com/grafana/tempo.git
cd tempo

# 2. Build Tempo
make tempo

# 3. Run local example
cd example/docker-compose/local
mkdir -p tempo-data
docker compose up -d

# 4. Access Grafana
open http://localhost:3000
# Login: admin/admin
```

## Essential Commands

### Building

```bash
make tempo              # Build main binary
make tempo-cli          # Build CLI tool
make test              # Run all tests
make lint              # Check code quality
make fmt               # Format code
```

### Testing

```bash
make test                      # All unit tests
make test-with-cover          # Tests with coverage
make test-e2e                 # End-to-end tests
make test-e2e-api            # API E2E tests only
```

### Docker

```bash
make docker-tempo             # Build Docker image
docker compose up -d          # Start services
docker compose logs tempo -f  # View Tempo logs
docker compose down -v        # Stop and clean up
```

## Project Structure (Essential Paths)

```
tempo/
├── cmd/tempo/              # Main application entry point
├── modules/
│   ├── distributor/       # Span ingestion
│   ├── ingester/          # Trace batching
│   ├── querier/           # Trace retrieval
│   └── frontend/          # Query coordination
├── tempodb/               # Storage layer
├── integration/e2e/       # End-to-end tests
└── example/docker-compose/ # Local examples
```

## Common Development Tasks

### Making a Code Change

```bash
# 1. Create a branch
git checkout -b feature/my-change

# 2. Make changes and format
# ... edit files ...
make fmt

# 3. Test changes
make lint
make test

# 4. Commit
git add .
git commit -m "Brief description of change"

# 5. Create PR on GitHub
```

### Running a Specific Test

```bash
# Run test in a specific package
go test ./modules/distributor/... -v

# Run a specific test function
go test ./modules/querier/... -run TestQueryTrace -v
```

### Debugging with Logs

```bash
# Enable debug logging in config
# Add to tempo.yaml:
# log_level: debug

# View logs
docker compose logs tempo -f --tail=100
```

## API Endpoints

When running locally:

- **Tempo API**: http://localhost:3200
- **Grafana UI**: http://localhost:3000
- **Prometheus**: http://localhost:9090

### Key API Calls

```bash
# Query a trace by ID
curl http://localhost:3200/api/traces/<trace-id>

# Search traces with TraceQL
curl 'http://localhost:3200/api/search?q={status=error}'

# Check health
curl http://localhost:3200/ready
curl http://localhost:3200/status/services
```

## TraceQL Examples

```traceql
# Find all error traces
{ status = error }

# Find slow requests
{ duration > 1s }

# Find specific service
{ resource.service.name = "frontend" }

# Complex query
{ .service.name = "api" && .http.status_code = 500 }
```

## Troubleshooting

### Build Fails

```bash
# Clean and rebuild
make clean
make vendor-check
make tempo
```

### Tests Fail

```bash
# Run with verbose output
make test 2>&1 | tee test.log
grep -i "fail" test.log
```

### Docker Issues

```bash
# Clean everything
docker compose down -v
docker system prune -f

# Rebuild images
docker compose build --no-cache
docker compose up -d
```

### Permission Issues (Local Example)

```bash
# Fix tempo-data directory permissions
sudo chown -R $(id -u):$(id -g) tempo-data/
chmod -R 755 tempo-data/
```

## Useful Resources

- **Full Onboarding Guide**: See [ONBOARDING.md](../../../ONBOARDING.md) in the root directory
- **Contributing Guide**: [CONTRIBUTING.md](../../../CONTRIBUTING.md)
- **Architecture**: [Architecture documentation](./operations/architecture.md)
- **API Docs**: https://grafana.com/docs/tempo/latest/api_docs/

## Next Steps

1. **Read the full onboarding guide**: [ONBOARDING.md](../../../ONBOARDING.md)
2. **Explore the examples**: Try different setups in `example/docker-compose/`
3. **Pick an issue**: Look for `good first issue` labels on GitHub
4. **Join the community**: Slack channel #tempo

## Quick Tips

- 💡 Always run `make fmt` before committing
- 💡 Use `make help` to see all available commands
- 💡 Check `example/docker-compose/` for different deployment scenarios
- 💡 Enable verbose logging with `log_level: debug` in config
- 💡 Use `tempo-cli` for debugging blocks and traces

---

**Need help?** Ask in [#tempo on Slack](https://grafana.slack.com/archives/C01D981PEE5)
