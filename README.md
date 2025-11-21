# Multi-Agent Earnings Analyzer

A comprehensive multi-agent system that analyzes company earnings data using AI agents with specialized capabilities, built with FastAPI and Vite + TypeScript.

## Features

- 🤖 Multi-agent system with specialized agents for earnings analysis
- 📊 Natural language queries to earnings insights using OpenAI or Anthropic
- 📁 Data import and management capabilities
- 🔍 Deep company earnings analysis with multiple perspectives
- 🔒 SQL injection protection and secure data handling
- ⚡ Fast development with Vite, uv, and FastAPI

## Prerequisites

- Python 3.10+
- uv (Python package manager)
- Node.js 18+
- Bun (or your preferred npm tool: npm, yarn, etc.)
- OpenAI API key and/or Anthropic API key

## Setup

### 1. Install Dependencies

```bash
# Backend
cd app/server
uv sync --all-extras

# Frontend
cd app/client
bun install
```

### 2. Environment Configuration

Set up your API keys in the server directory:

```bash
cd app/server
cp .env.sample .env
# Edit .env and add your API keys
```

## Quick Start

Use the provided script to start both services:

```bash
./scripts/start.sh
```

Press `Ctrl+C` to stop both services.

The script will:
- Check that `.env` exists in `app/server/`
- Start the backend on http://localhost:8000
- Start the frontend on http://localhost:5173
- Handle graceful shutdown when you exit

## Manual Start (Alternative)

### Backend
```bash
cd app/server
# .env is loaded automatically by python-dotenv
uv run python server.py
```

### Frontend
```bash
cd app/client
bun run dev
```

## Usage

### Analyzing Earnings Reports

1. **Prepare Your Report**: Place your earnings report in the `data/` directory
   - Supported format: Plain text (.txt) files
   - Example: `data/earnings_report_sample.txt`

2. **Run Analysis**: Submit the report path to the API
   ```bash
   curl -X POST http://localhost:8000/analyze \
     -H "Content-Type: application/json" \
     -d '{"report_path": "data/earnings_report_sample.txt"}'
   ```

3. **View Results**: The system returns comprehensive analysis including:
   - **Financial Metrics**: Revenue, net income, EPS, operating margins, cash flow
   - **Segment Performance**: Revenue and growth rates by business segment
   - **Forward Guidance**: Projected metrics and growth targets
   - **Sentiment Analysis**: Overall tone, key indicators, and identified risks
   - **Executive Summary**: Investment recommendation (BUY/HOLD/SELL) with confidence score

### Interactive API Testing

Use the Swagger UI for interactive testing:
- Visit: http://localhost:8000/docs
- Use the built-in "Try it out" feature for each endpoint
- View real-time responses and API documentation

## Development

### Backend Commands
```bash
cd app/server
uv run python server.py      # Start server with hot reload
uv run pytest               # Run tests
uv add <package>            # Add package to project
uv remove <package>         # Remove package from project
uv sync --all-extras        # Sync all extras
```

### Frontend Commands
```bash
cd app/client
bun run dev                 # Start dev server
bun run build              # Build for production
bun run preview            # Preview production build
```

## Project Structure

```
.
├── app/                    # Main application
│   ├── client/             # Vite + TypeScript frontend
│   └── server/             # FastAPI backend
│
├── adws/                   # AI Developer Workflow (ADW) - GitHub issue automation system
├── scripts/                # Utility scripts (start.sh, stop_apps.sh)
├── specs/                  # Feature specifications
├── ai_docs/                # AI/LLM documentation
├── agents/                 # Agent execution logging
└── logs/                   # Structured session logs
```

## API Endpoints

- `GET /` - API information and version
- `GET /health` - Health check and agent status
- `GET /agents` - List all available agents
- `POST /analyze` - Analyze earnings report
  - **Input**: `{"report_path": "path/to/report.txt"}`
  - **Output**: Complete financial analysis with multi-agent insights
- `GET /docs` - Interactive Swagger UI documentation
- `GET /redoc` - ReDoc API documentation

## Security

### Multi-Agent System Safety

The multi-agent earnings analyzer implements several security and reliability measures:

1. **Input Validation**:
   - File path validation to ensure only valid report files are processed
   - Report content validation before analysis
   - Graceful error handling for missing or malformed files

2. **Agent Isolation**:
   - Each agent operates independently with its own state
   - No cross-contamination between analysis runs
   - Agents validate input data before processing

3. **API Security**:
   - CORS configured for secure cross-origin requests
   - All API endpoints validate request payloads
   - Error responses are sanitized to avoid information leakage

4. **Data Handling**:
   - No sensitive data is stored or cached between requests
   - Each analysis request is processed in isolation
   - LLM interactions use secure API key authentication

### LLM Integration Security

- API keys are loaded from environment variables (never hardcoded)
- Supports both Anthropic Claude and Mock LLM for testing
- Failed LLM calls are handled gracefully with fallback logic
- All LLM tokens and costs are logged for monitoring

### Best Practices for Deployment

1. Set `ANTHROPIC_API_KEY` environment variable securely
2. Use HTTPS in production deployments
3. Implement API rate limiting for production use
4. Monitor token usage and costs
5. Add authentication/authorization layers as needed
6. Maintain audit logs of all analysis requests

## AI Developer Workflow (ADW)

The ADW system is a comprehensive automation framework that integrates GitHub issues with Claude Code CLI to classify issues, generate implementation plans, and automatically create pull requests. ADW processes GitHub issues by classifying them as `/chore`, `/bug`, or `/feature` commands and then implementing solutions autonomously.

### Prerequisites

Before using ADW, ensure you have the following installed and configured:

- **GitHub CLI**: `brew install gh` (macOS) or equivalent for your OS
- **Claude Code CLI**: Install from [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code)
- **Python with uv**: `curl -LsSf https://astral.sh/uv/install.sh | sh`
- **GitHub authentication**: `gh auth login`

### Environment Variables

Set these environment variables before running ADW:

```bash
export GITHUB_REPO_URL="https://github.com/owner/repository"
export ANTHROPIC_API_KEY="sk-ant-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
export CLAUDE_CODE_PATH="/path/to/claude"  # Optional, defaults to "claude"
export GITHUB_PAT="ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"  # Optional, only if using different account than 'gh auth login'
```

### Usage Modes

ADW supports three main operation modes:

#### 1. Manual Processing
Process a single GitHub issue manually:
```bash
cd adws/
uv run adw_plan_build.py <issue-number>
```

#### 2. Automated Monitoring
Continuously monitor GitHub for new issues (polls every 20 seconds):
```bash
cd adws/
uv run trigger_cron.py
```

#### 3. Webhook Server
Start a webhook server for real-time GitHub event processing:
```bash
cd adws/
uv run trigger_webhook.py
```

### How ADW Works

1. **Issue Classification**: Analyzes GitHub issues and determines type (`/chore`, `/bug`, `/feature`)
2. **Planning**: Generates detailed implementation plans using Claude Code CLI
3. **Implementation**: Executes the plan by making code changes, running tests, and ensuring quality
4. **Integration**: Creates git commits and pull requests with semantic commit messages

### For More Information

For detailed technical documentation, configuration options, and troubleshooting, see [`adws/README.md`](adws/README.md).

## Troubleshooting

**Backend won't start:**
- Check Python version: `python --version` (requires 3.12+)
- Verify API keys are set: `echo $OPENAI_API_KEY`

**Frontend errors:**
- Clear node_modules: `rm -rf node_modules && bun install`
- Check Node version: `node --version` (requires 18+)

**CORS issues:**
- Ensure backend is running on port 8000
- Check vite.config.ts proxy settings# multi-agent-workflow-company-earnings
