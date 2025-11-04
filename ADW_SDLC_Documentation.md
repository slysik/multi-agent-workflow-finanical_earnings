# ADW SDLC Agentic Data Flow - Complete Guide

## Overview

The ADW (AI Developer Workflow) SDLC system is an **autonomous software development lifecycle orchestrator** that uses Claude Code agents to execute the complete development workflow from planning through documentation. It manages a multi-phase pipeline with persistent state management and agent coordination.

---

## Core Architecture

### Main Entry Point: `adw_sdlc.py`

**Purpose**: Orchestrates the complete 5-phase SDLC workflow

**Input**:
- `issue-number` (required): GitHub issue number
- `adw-id` (optional): Unique workflow identifier (auto-generated if not provided)

**Output**: Complete development workflow execution with persistent state tracking

---

## The Five SDLC Phases

### Phase 1: Planning (`adw_plan.py`)
- Fetches GitHub issue details
- Classifies issue type (/chore, /bug, /feature)
- Creates feature branch
- Generates implementation plan
- Commits plan to repository

### Phase 2: Building (`adw_build.py`)
- Implements the plan from Phase 1
- Executes code changes
- Creates working implementation

### Phase 3: Testing (`adw_test.py`)
- Runs automated tests (skips E2E by default)
- Validates implementation
- Reports test results

### Phase 4: Review (`adw_review.py`)
- Reviews code changes
- Checks quality and standards
- Identifies issues or improvements

### Phase 5: Documentation (`adw_document.py`)
- Generates documentation
- Updates README/docs
- Creates final documentation artifacts

---

## Key Components

### 1. State Management (`adw_modules/state.py`)

**ADWState Class** - Persistent state container

**Core State Fields**:
- `adw_id`: Unique workflow identifier
- `issue_number`: GitHub issue number
- `branch_name`: Git branch name
- `plan_file`: Path to plan specification
- `issue_class`: Issue type (/chore, /bug, /feature)

**Storage Location**: `agents/{adw_id}/adw_state.json`

**Key Methods**:
- `save()`: Persists state to JSON file
- `load()`: Loads existing state
- `update()`: Updates state fields
- `get()`: Retrieves state values

---

### 2. Agent Execution (`adw_modules/agent.py`)

**Purpose**: Executes Claude Code agents programmatically

**Key Functions**:

#### `execute_template(request: AgentTemplateRequest)`
- Executes slash commands with agent coordination
- Handles model selection based on task complexity
- Manages output and logging

#### `prompt_claude_code(request: AgentPromptRequest)`
- Low-level Claude Code CLI execution
- Parses JSONL output streams
- Handles errors and timeouts

**Agent Names** (from `workflow_ops.py`):
- `sdlc_planner`: Planning agent
- `sdlc_implementor`: Implementation agent
- `issue_classifier`: Issue type classifier
- `branch_generator`: Branch name generator
- `pr_creator`: Pull request creator

**Model Selection Strategy**:
```
Simple tasks → sonnet (fast, cost-effective)
Complex tasks → opus (powerful, thorough)
```

---

### 3. Workflow Operations (`adw_modules/workflow_ops.py`)

**High-level orchestration functions**:

#### Core Operations

**`ensure_adw_id(issue_number, adw_id)`**
- Creates or retrieves ADW ID
- Initializes state if needed
- Returns valid ADW ID

**`classify_issue(issue, adw_id, logger)`**
- Uses `issue_classifier` agent
- Returns: `/chore`, `/bug`, or `/feature`
- Determines workflow path

**`build_plan(issue, command, adw_id, logger)`**
- Uses `sdlc_planner` agent
- Generates implementation specification
- Returns plan file path

**`implement_plan(plan_file, adw_id, logger)`**
- Uses `sdlc_implementor` agent
- Executes the plan
- Creates implementation

**`generate_branch_name(issue, issue_class, adw_id, logger)`**
- Uses `branch_generator` agent
- Creates standardized branch name
- Format: `{type}-issue-{number}-adw-{adw_id}-{description}`

**`create_commit(agent_name, issue, issue_class, adw_id, logger)`**
- Creates formatted commit message
- Uses agent-specific committer
- Returns commit message

**`create_pull_request(branch_name, issue, state, logger)`**
- Uses `pr_creator` agent
- Creates GitHub pull request
- Returns PR URL

---

## Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    adw_sdlc.py (Main)                        │
│                                                               │
│  Input: issue_number, adw_id (optional)                      │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
          ┌──────────────────────┐
          │ ensure_adw_id()      │ ← Creates/retrieves workflow ID
          │ ADWState initialized  │
          └──────────┬───────────┘
                     │
     ┌───────────────┴───────────────┐
     │    State: adw_state.json       │
     │    Location: agents/{adw_id}/  │
     └───────────────┬───────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼
  ┌─────────┐              ┌──────────┐
  │  PHASE  │              │  Agents  │
  │ Scripts │◄────────────►│  System  │
  └────┬────┘              └─────┬────┘
       │                         │
       │  ┌──────────────────────┤
       │  │                      │
       ▼  ▼                      ▼
    ┌────────────┐      ┌────────────────┐
    │ State      │      │ Claude Code    │
    │ Management │      │ CLI Execution  │
    └────────────┘      └────────────────┘
```

---

## Detailed Phase Flow

### Phase Execution Pattern (Each Phase):

```
1. Load State
   ├─► ADWState.load(adw_id)
   └─► Retrieve previous phase outputs

2. Execute Agent Operations
   ├─► AgentTemplateRequest created
   ├─► execute_template() called
   ├─► Claude Code CLI invoked
   └─► Output parsed from JSONL

3. Update State
   ├─► state.update(new_fields)
   └─► state.save(workflow_step)

4. Git Operations (if applicable)
   ├─► Branch management
   ├─► Commits
   └─► Push operations

5. Return Control
   └─► Exit code (0=success, 1=failure)
```

---

## Agent Communication Flow

```
┌──────────────────┐
│ Workflow Script  │
│  (e.g. adw_plan) │
└────────┬─────────┘
         │
         │ 1. Create AgentTemplateRequest
         ▼
┌──────────────────────┐
│ execute_template()   │
│  - Select model      │
│  - Build prompt      │
│  - Set output path   │
└────────┬─────────────┘
         │
         │ 2. Create AgentPromptRequest
         ▼
┌──────────────────────┐
│ prompt_claude_code() │
│  - Save prompt       │
│  - Run CLI           │
│  - Parse JSONL       │
└────────┬─────────────┘
         │
         │ 3. Execute via subprocess
         ▼
┌──────────────────────┐
│  Claude Code CLI     │
│  - Process prompt    │
│  - Stream output     │
│  - Return results    │
└────────┬─────────────┘
         │
         │ 4. Parse & Return
         ▼
┌──────────────────────┐
│ AgentPromptResponse  │
│  - output: str       │
│  - success: bool     │
│  - session_id: str   │
└──────────────────────┘
```

---

## File System Layout

```
project_root/
├── adws/
│   ├── adw_sdlc.py          # Main orchestrator
│   ├── adw_plan.py          # Phase 1
│   ├── adw_build.py         # Phase 2
│   ├── adw_test.py          # Phase 3
│   ├── adw_review.py        # Phase 4
│   ├── adw_document.py      # Phase 5
│   └── adw_modules/
│       ├── state.py         # State management
│       ├── agent.py         # Agent execution
│       ├── workflow_ops.py  # High-level operations
│       ├── git_ops.py       # Git operations
│       ├── github.py        # GitHub API
│       └── data_types.py    # Type definitions
│
└── agents/
    └── {adw_id}/            # Per-workflow directory
        ├── adw_state.json   # Persistent state
        ├── sdlc_planner/
        │   ├── prompts/
        │   ├── raw_output.jsonl
        │   └── raw_output.json
        ├── sdlc_implementor/
        │   └── ...
        └── ... (other agents)
```

---

## Key Data Types

### AgentTemplateRequest
```python
- agent_name: str          # Which agent to use
- slash_command: str       # Command to execute
- args: List[str]          # Command arguments
- adw_id: str             # Workflow identifier
- model: str              # Claude model (sonnet/opus)
```

### AgentPromptResponse
```python
- output: str             # Agent's response
- success: bool           # Execution success
- session_id: str         # Session identifier
```

### ADWStateData
```python
- adw_id: str            # Unique workflow ID
- issue_number: str      # GitHub issue number
- branch_name: str       # Git branch
- plan_file: str         # Spec file path
- issue_class: str       # Issue type
```

---

## Usage Examples

### Basic Usage
```bash
# Run complete SDLC for issue #42
uv run adws/adw_sdlc.py 42
```

### With Existing ADW ID
```bash
# Continue workflow with specific ID
uv run adws/adw_sdlc.py 42 abc123def456
```

### Individual Phases
```bash
# Run only planning phase
uv run adws/adw_plan.py 42

# Run only build phase (requires existing plan)
uv run adws/adw_build.py 42 abc123def456
```

---

## Error Handling & Recovery

### State Persistence
- State saved after each operation
- Allows recovery from failures
- Can resume at any phase

### Phase Independence
- Each phase can run standalone
- Phases check for required state
- Error in one phase doesn't corrupt others

### Validation
- Environment variable checks
- State validation with Pydantic
- Claude Code CLI availability checks

---

## Benefits of This Architecture

1. **Modularity**: Each phase is independent and reusable
2. **Persistence**: State survives crashes and allows resume
3. **Traceability**: All agent interactions logged
4. **Flexibility**: Can run full pipeline or individual phases
5. **Scalability**: Multiple workflows can run concurrently with different ADW IDs
6. **Type Safety**: Pydantic models ensure data integrity
7. **Agent Specialization**: Different agents for different tasks optimize performance and cost

---

## Summary

The ADW SDLC system creates a **fully autonomous development pipeline** where:

1. **Input**: GitHub issue number
2. **Process**: Five automated phases (Plan → Build → Test → Review → Document)
3. **Agents**: Specialized Claude Code agents for each task
4. **State**: Persistent tracking across all phases
5. **Output**: Complete implementation with PR and documentation

Each phase communicates through:
- **Persistent state files** (JSON)
- **Agent execution system** (Claude Code CLI)
- **Git operations** (branches, commits, PRs)
- **GitHub integration** (issues, comments, PRs)

This creates a **composable, resilient, and traceable** development workflow that can handle complex software development tasks autonomously.
