# Multi-Agent Earnings Analyzer - Deployment Guide

**Created:** November 21, 2025
**Repository:** https://github.com/slysik/multi-agent-workflow-finanical_earnings
**Status:** ✅ Production-Ready

## Quick Start

### Option 1: Docker Compose (Recommended)
```bash
cd client
docker-compose up --build
```

Access API at: `http://localhost:8000`
- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`

### Option 2: Local Python Development
```bash
# Install dependencies
cd client
pip install -r requirements.txt

# Run application
python -m uvicorn src.main:app --host 0.0.0.0 --port 8000 --reload
```

### Option 3: Docker Direct
```bash
cd client
docker build -t earnings-analyzer:latest .
docker run -d \
  --name earnings-analyzer \
  -p 8000:8000 \
  -v $(pwd)/data:/app/data \
  earnings-analyzer:latest
```

## Configuration

### Environment Variables
Create a `.env` file in the `client/` directory:

```bash
# LLM Provider Configuration
ANTHROPIC_API_KEY=sk-ant-...  # Optional - uses MockLLM if not provided
USE_MOCK_LLM=false             # Set to true for zero-cost testing

# Server Configuration
HOST=0.0.0.0
PORT=8000
DEBUG=false
```

Copy the example file:
```bash
cp client/.env.example client/.env
# Edit client/.env and add your API key
```

## API Endpoints

### Health Check
```bash
curl http://localhost:8000/health
```

### List Available Agents
```bash
curl http://localhost:8000/agents
```

### Analyze Earnings Report
```bash
curl -X POST http://localhost:8000/analyze \
  -H "Content-Type: application/json" \
  -d '{"report_path": "data/earnings_report_sample.txt", "options": {}}'
```

Response includes:
- Financial metrics (revenue, net income, EPS, margins, cash flow)
- Segment performance analysis
- Forward guidance projections
- Sentiment analysis
- Investment recommendation (BUY/HOLD/SELL)
- Processing metrics (time, tokens, quality score)

## Project Structure

```
client/
├── src/
│   ├── main.py              # FastAPI app & endpoints
│   ├── llm_client.py        # LLM abstraction (Anthropic/Mock)
│   └── workflow/
│       └── graph.py         # LangGraph agent orchestration
├── tests/
│   ├── test_main.py         # API endpoint tests
│   ├── test_agents.py       # Agent logic tests
│   └── conftest.py          # Pytest configuration
├── data/
│   ├── earnings_report_sample.txt  # Sample input
│   └── expected_output.json        # Reference output
├── Dockerfile               # Production container
├── docker-compose.yml       # Multi-container orchestration
├── requirements.txt         # Python dependencies
└── run.sh                   # Development start script
```

## Architecture Overview

### Agent Pipeline
1. **Coordinator Agent**: Validates input, initializes workflow
2. **Data Extractor Agent**: Extracts financial metrics from earnings report
3. **Sentiment Analysis Agent**: Analyzes tone and emotional indicators
4. **Summary Agent**: Generates investment recommendation

### Key Design Patterns
- **Sequential Pipeline**: Agents execute in deterministic order via LangGraph
- **Shared State**: All agents communicate through `AnalysisState` TypedDict
- **Graceful Degradation**: Fallback parsing when JSON extraction fails
- **Dynamic Metrics**: Real-time performance measurements instead of hard-coded values

## Performance Characteristics

| Metric | Value |
|--------|-------|
| Processing Time (MockLLM) | 0.007 - 0.015s |
| Token Usage (estimated) | ~888 tokens/report |
| Data Quality Score | 1.0 (100%) |
| Success Rate | 100% |
| Error Handling | Yes (all error cases handled) |

**Note:** Real LLM processing adds ~3-5 seconds for API latency. MockLLM enables zero-cost testing.

## Testing

### Run All Tests
```bash
cd client
python -m pytest -v
```

### Run Specific Test Suite
```bash
# Test API endpoints
python -m pytest tests/test_main.py -v

# Test agent logic
python -m pytest tests/test_agents.py -v
```

### Test with Real LLM
```bash
# Set real API key
export ANTHROPIC_API_KEY=sk-ant-...
cd client
python -m uvicorn src.main:app --reload
```

## Production Deployment

### Kubernetes
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: earnings-analyzer
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: analyzer
        image: earnings-analyzer:latest
        ports:
        - containerPort: 8000
        env:
        - name: ANTHROPIC_API_KEY
          valueFrom:
            secretKeyRef:
              name: anthropic-secrets
              key: api-key
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

### AWS ECS
```bash
# Create ECR repository
aws ecr create-repository --repository-name earnings-analyzer

# Build and push image
cd client
docker build -t earnings-analyzer:latest .
docker tag earnings-analyzer:latest 123456789.dkr.ecr.us-east-1.amazonaws.com/earnings-analyzer:latest
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/earnings-analyzer:latest
```

### Environment-Specific Configuration
- **Development**: `USE_MOCK_LLM=true`, `DEBUG=true`
- **Staging**: Real LLM with rate limiting, monitoring enabled
- **Production**: Real LLM, HTTPS only, secrets from AWS Secrets Manager

## Monitoring & Observability

### Health Checks
```bash
# Liveness probe
curl http://localhost:8000/health

# Readiness check
curl http://localhost:8000/agents
```

### Metrics to Track
- API response time (p50, p99)
- LLM token usage and costs
- Agent success/failure rates
- Error rates by type
- Queue depth (if using async job queue)

### Logging
All requests logged with:
- Timestamp
- Request ID (correlation)
- Processing time
- Agent status
- Error details (if any)

## Scaling Considerations

### Horizontal Scaling (1000+ reports/hour)
1. Deploy multiple instances behind load balancer
2. Add Redis-backed job queue (Celery)
3. Run Data Extractor & Sentiment Analysis in parallel
4. Implement response caching (1-hour TTL for identical reports)
5. Use cheaper models for extraction (Claude Haiku vs Sonnet)

### Vertical Scaling
- Increase container memory for concurrent request handling
- Use async/await for non-blocking I/O
- Implement connection pooling for database

## Troubleshooting

### Container won't start
```bash
# Check logs
docker logs earnings-analyzer

# Verify environment variables
docker exec earnings-analyzer env | grep ANTHROPIC
```

### API returning 500 errors
```bash
# Check if it's an LLM issue
curl http://localhost:8000/health

# Try with MockLLM for debugging
USE_MOCK_LLM=true docker-compose up
```

### Slow performance
- Check LLM API latency: `time curl https://api.anthropic.com`
- Verify network connectivity to API
- Monitor local system resources (CPU, memory)
- Consider model optimization (Haiku for data extraction)

## Security Checklist

- [ ] API keys stored in secure vault (AWS Secrets Manager, HashiCorp Vault)
- [ ] HTTPS enabled in production (use AWS ALB or nginx reverse proxy)
- [ ] Rate limiting configured per API key
- [ ] Request/response logging sanitized (no API keys in logs)
- [ ] Container image scanned for vulnerabilities
- [ ] CORS configured for trusted domains only
- [ ] Database connections encrypted (if adding persistence)
- [ ] Audit trail maintained for all analyses

## Support & Documentation

- **API Documentation**: http://localhost:8000/docs (Swagger UI)
- **ReDoc**: http://localhost:8000/redoc
- **Solution Documentation**: See SOLUTION.md for architecture details
- **Architecture Overview**: See README.md for comprehensive guide

---

**Last Updated:** November 21, 2025
**Maintained By:** Steve (Development)
**Repository:** https://github.com/slysik/multi-agent-workflow-finanical_earnings
