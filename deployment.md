# Docker Deployment Guide

**Generated:** 2026-03-10  
**Target:** Wave 3+4 UC Services + Supporting Infrastructure

## Prerequisites

- Docker 20.10+
- Docker Compose 2.0+
- 8GB+ RAM (recommended for running all services)
- Ports 8001-8009 available (UC services)
- Optional: Ports 4000, 8080, 8200, 9000 (supporting services)

## Quick Start

```bash
# Clone the repository
cd ai-native-agentic-org

# Start all services
docker-compose up -d

# Verify all services are healthy
docker-compose ps

# View logs
docker-compose logs -f

# Stop all services
docker-compose down
```

## Port Mapping

| Service | Port | Health Check Endpoint | Purpose |
|---------|------|----------------------|---------|
| agent-registry-service | 8001 | http://localhost:8001/health | Agent discovery and capability registry |
| enterprise-gateway | 8003 | http://localhost:8003/health | Enterprise SSO and policy enforcement |
| multichannel-support-platform | 8004 | http://localhost:8004/health | AI support responder + channel bridges |
| ai-gig-marketplace | 8005 | http://localhost:8005/health | Real-time gig auction platform |
| ondevice-ai-assistant | 8006 | http://localhost:8006/health | On-device inference with Ollama |
| self-evolving-economic-agent | 8007 | http://localhost:8007/health | DSPy self-learner with RL |
| multi-agent-hub | 8008 | http://localhost:8008/health | DAG workflow orchestration |
| cross-project-hub | 8009 | http://localhost:8009/health | Cross-service integration hub |

### Supporting Services (Optional)

| Service | Port | Purpose |
|---------|------|---------|
| symphony | 4000 | Elixir OTP orchestrator |
| openmanus | 8080 | Computer use agent |
| nanobot | 9000 | Lightweight agent |
| clawwork | 8200 | Economic benchmark |

## Environment Variables

### Global Configuration

All services support:

```bash
MOCK_MODE=true  # Enable mock mode for testing without external dependencies
```

### UC9: Cross-Project Hub

UC9 requires service discovery URLs:

```bash
# Service URLs (internal Docker network)
KOREAN_BOT_URL=http://korean-service-bot:8002
MARKETPLACE_URL=http://ai-gig-marketplace:8005
REGISTRY_URL=http://agent-registry-service:8001
HUB_URL=http://multi-agent-hub:8008
ONDEVICE_URL=http://ondevice-ai-assistant:8006
SELF_EVOLVING_URL=http://self-evolving-economic-agent:8007
MULTICHANNEL_URL=http://multichannel-support-platform:8004
ENTERPRISE_URL=http://enterprise-gateway:8003
```

### UC8: Multi-Agent Hub

```bash
SYMPHONY_URL=http://localhost:4000/api
OPENMANUS_URL=http://localhost:8080/a2a
NANOBOT_URL=http://localhost:9000/api
MOCK_MODE=true
```

### UC5: AI Gig Marketplace

```bash
PLATFORM_FEE_PCT=0.15
MIN_QUALITY_SCORE=0.70
CLAWWORK_API_URL=http://localhost:8200/api
MAX_BID_MULTIPLIER=3.0
```

### UC6: Ondevice AI Assistant

```bash
OLLAMA_BASE_URL=http://localhost:11434
DEFAULT_MODEL=llama2
MOCK_MODE=true  # Falls back to mock responses if Ollama unavailable
```

## Health Check Verification

All services include health checks with the following configuration:

- **Interval**: 10 seconds
- **Timeout**: 5 seconds
- **Retries**: 3
- **Start period**: 10 seconds

### Manual Health Check

```bash
# Check individual service
curl -sf http://localhost:8001/health && echo "✓ UC1 healthy"
curl -sf http://localhost:8003/health && echo "✓ UC3 healthy"
curl -sf http://localhost:8004/health && echo "✓ UC4 healthy"
curl -sf http://localhost:8005/health && echo "✓ UC5 healthy"
curl -sf http://localhost:8006/health && echo "✓ UC6 healthy"
curl -sf http://localhost:8007/health && echo "✓ UC7 healthy"
curl -sf http://localhost:8008/health && echo "✓ UC8 healthy"
curl -sf http://localhost:8009/health && echo "✓ UC9 healthy"

# Check all services at once
for port in 8001 8003 8004 8005 8006 8007 8008 8009; do
  curl -sf http://localhost:$port/health > /dev/null && echo "✓ Port $port healthy" || echo "✗ Port $port unhealthy"
done
```

## Service Dependencies

UC9 (Cross-Project Hub) depends on all other services being healthy before starting. Docker Compose enforces this startup order:

1. UC1, UC3, UC4, UC5, UC6, UC7, UC8 start in parallel
2. Health checks run every 10s
3. UC9 starts only after all dependencies are healthy

## API Documentation

Each service exposes OpenAPI documentation:

- UC1: http://localhost:8001/docs
- UC3: http://localhost:8003/docs
- UC4: http://localhost:8004/docs
- UC5: http://localhost:8005/docs
- UC6: http://localhost:8006/docs
- UC7: http://localhost:8007/docs
- UC8: http://localhost:8008/docs
- UC9: http://localhost:8009/docs

## Troubleshooting

### Service Won't Start

```bash
# Check logs for specific service
docker-compose logs agent-registry-service
docker-compose logs cross-project-hub

# Restart specific service
docker-compose restart agent-registry-service

# Rebuild and restart
docker-compose up -d --build agent-registry-service
```

### Port Already in Use

```bash
# Find process using port
lsof -i :8001
netstat -tulpn | grep 8001

# Kill process or change port in docker-compose.yml
```

### Health Check Failing

```bash
# Check service logs
docker-compose logs -f multichannel-support-platform

# Verify service is responding
docker-compose exec multichannel-support-platform curl http://localhost:8004/health

# Check container status
docker-compose ps
```

### UC9 Won't Start (Dependency Issues)

UC9 requires all other services to be healthy. Check dependencies:

```bash
# Verify all dependencies are healthy
docker-compose ps

# Check UC9 logs for specific failure
docker-compose logs cross-project-hub

# Restart dependencies first
docker-compose restart agent-registry-service ai-gig-marketplace
docker-compose restart cross-project-hub
```

### Out of Memory

```bash
# Check container memory usage
docker stats

# Increase Docker memory limit (Docker Desktop: Settings → Resources)
# Or stop non-essential services
docker-compose stop korean-service-bot  # Out of scope for Wave 3
```

### Network Issues

```bash
# Verify Docker network
docker network ls
docker network inspect ai-native-agentic-org_default

# Recreate network
docker-compose down
docker-compose up -d
```

## Production Deployment

### Disable Mock Mode

```bash
# Edit docker-compose.yml or use environment override
MOCK_MODE=false docker-compose up -d
```

### External Service Configuration

For production, configure external dependencies:

```bash
# UC6: Ollama backend
OLLAMA_BASE_URL=http://ollama-server:11434

# UC5: ClawWork API
CLAWWORK_API_URL=http://clawwork-server:8200/api

# UC8: External agent bridges
SYMPHONY_URL=http://symphony-server:4000/api
OPENMANUS_URL=http://openmanus-server:8080/a2a
NANOBOT_URL=http://nanobot-server:9000/api
```

### Monitoring

```bash
# Prometheus metrics (if enabled)
curl http://localhost:8009/metrics

# Health check aggregation
curl http://localhost:8009/health/all
```

## Development Workflow

### Local Development

```bash
# Start only required services
docker-compose up -d agent-registry-service ai-gig-marketplace

# Run service locally
cd multichannel-support-platform
uvicorn multichannel_support_platform.main:app --reload --port 8004

# Run tests
pytest tests/
```

### Rebuild After Code Changes

```bash
# Rebuild specific service
docker-compose build multichannel-support-platform
docker-compose up -d multichannel-support-platform

# Rebuild all services
docker-compose build
docker-compose up -d
```

## Testing

### Integration Tests

```bash
# Run tests for specific service
docker-compose exec multichannel-support-platform pytest tests/

# Run all tests
for service in agent-registry-service enterprise-gateway multichannel-support-platform ai-gig-marketplace ondevice-ai-assistant self-evolving-economic-agent multi-agent-hub cross-project-hub; do
  echo "Testing $service..."
  docker-compose exec $service pytest tests/ || echo "✗ $service tests failed"
done
```

### Load Testing

```bash
# UC5: Gig marketplace auction
ab -n 1000 -c 10 http://localhost:8005/tasks

# UC4: Support ticket creation
ab -n 1000 -c 10 -p ticket.json -T application/json http://localhost:8004/tickets
```

## Notes

- **UC1 and UC3** are production-ready services with full Docker and mock mode support
- **UC2 (korean-service-bot, port 8002)** exists in docker-compose but is out of scope
- All Python services use FastAPI with async/await patterns
- Health checks enforce proper startup order via `depends_on` with `condition: service_healthy`
- Services communicate via internal Docker network using service names as hostnames
- External access uses localhost with mapped ports (8001-8009)

## Quick Reference

```bash
# Start all services
docker-compose up -d

# Stop all services
docker-compose down

# View logs
docker-compose logs -f [service-name]

# Restart service
docker-compose restart [service-name]

# Rebuild service
docker-compose build [service-name]

# Check status
docker-compose ps

# Run tests
docker-compose exec [service-name] pytest tests/

# Access shell
docker-compose exec [service-name] /bin/bash
```
