# VSCode Ralph Loop Agents

A language-agnostic implementation of the Ralph Wiggum autonomous agent loop for VSCode, using custom agents with automatic handoffs.

## What is Ralph Loop?

**Ralph loop = Fresh context + Filesystem memory**

An autonomous coding pattern where:
1. Agent picks a task from PRD
2. Executes it with fresh context
3. Commits and updates progress
4. Loops until all tasks complete

Named after Ralph Wiggum from The Simpsons - persistent iteration beats one-shot perfection.

## Architecture

```
┌──────────┐
│  Human   │ Creates requirements
└────┬─────┘
     │
     ▼
┌──────────────┐  Generates PRD
│   Planner    ├──────────┐
└──────────────┘          │
                          ▼
                    ┌─────────┐
                    │ PRD.md  │
                    └────┬────┘
                         │
    ┌────────────────────┴────────────────────┐
    │                                         │
    ▼                                         ▼
┌─────────────┐  Assigns task  ┌──────────────┐
│ Coordinator │◄───────────────┤   Executor   │
│  (Agent 1)  │                │  (Agent 2)   │
└──────┬──────┘                └──────┬───────┘
       │                              │
       │  Handoff ─────────────────►  │
       │                              │
       │  ◄───────── Handoff          │
       │                              │
       └──────────┐          ┌────────┘
                  ▼          ▼
            ┌──────────────────┐
            │   PROGRESS.md    │
            │   (Filesystem    │
            │     Memory)      │
            └──────────────────┘
```

## Features

✅ **Automatic handoffs** - Agents pass control automatically
✅ **Progress file** - Fresh context every iteration via PROGRESS.md
✅ **Language agnostic** - Works with any programming language/stack
✅ **Atomic tasks** - One task per iteration, committed immediately
✅ **Context reset** - Avoids context pollution, uses filesystem as memory

## Setup

### Prerequisites

- VSCode 1.106+ (custom agents support)
- Git repository initialized in your project
- VSCode Copilot or compatible AI extension

### Installation

1. Copy agent files to your project:

```bash
mkdir -p .github/agents
cp coordinator.agent.md .github/agents/
cp executor.agent.md .github/agents/
cp planner.agent.md .github/agents/
```

2. Restart VSCode or reload window

3. Verify agents are available:
   - Open Command Palette (`Cmd+Shift+P` / `Ctrl+Shift+P`)
   - Type "Select Agent"
   - Should see: Coordinator, Executor, Planner

## Usage

### Quick Start

1. **Create PRD** with Planner agent:
   ```
   Open VSCode Chat
   Select: Planner agent
   Prompt: "Create a PRD for [your feature]"
   ```

2. **Review PRD.md** - Edit as needed

3. **Start Loop** with Coordinator:
   ```
   Select: Coordinator agent
   Click: "Start Ralph Loop" handoff button
   ```

4. **Let it run** - Agents will:
   - Pick tasks from PRD.md
   - Execute them
   - Update PROGRESS.md
   - Commit changes
   - Repeat until done

5. **Monitor progress**:
   ```bash
   cat PROGRESS.md
   git log --oneline -10
   ```

### Manual Iteration (Human-in-the-Loop)

For controlled execution:

1. Coordinator selects task
2. Click "Execute Task" handoff
3. Executor completes task
4. Review changes
5. Click "Continue Loop" handoff
6. Repeat

### Fully Autonomous

For overnight/background execution:

1. Set `send: true` in handoff definitions (already configured)
2. Start Coordinator
3. Walk away
4. Return to completed work

**Note**: Monitor costs and set iteration limits in PRD.

## File Structure

```
your-project/
├── .github/
│   └── agents/
│       ├── coordinator.agent.md    # Task orchestrator
│       ├── executor.agent.md       # Task implementer
│       └── planner.agent.md        # PRD creator
├── PRD.md                          # Product requirements
├── PROGRESS.md                     # Iteration state
├── src/                            # Your code
└── tests/                          # Your tests
```

## PROGRESS.md Format

```markdown
# Progress Log

## Completed
- [x] Task-001: Setup (commit: a1b2c3d)
- [x] Task-002: Auth (commit: e4f5g6h)

## Current Iteration
- Iteration: 5
- Working on: Task-003
- Started: 2026-01-30T10:00:00Z

## Last Completed
- Task-002: Authentication
- Duration: 15 min
- Tests: ✅ All passing
- Key decisions: Using JWT

## Blockers
- None

## Notes
- Auth middleware is in middleware/auth.js
```

## PRD.md Format

```markdown
# Feature: User Authentication

## Tasks

### Task-001: Setup JWT Library
**Acceptance Criteria**:
- [ ] jsonwebtoken installed
- [ ] Config file created
- [ ] Environment variables set

**Verification**:
```bash
npm test
```

### Task-002: Login Endpoint
**Acceptance Criteria**:
- [ ] POST /auth/login route
- [ ] Password validation
- [ ] JWT token returned
- [ ] Tests passing

**Verification**:
```bash
npm test -- auth.test.js
```
```

## Language Examples

### Python
```markdown
**Verification**:
```bash
pytest tests/
black --check .
mypy .
```
```

### Rust
```markdown
**Verification**:
```bash
cargo test
cargo clippy
cargo build --release
```
```

### Go
```markdown
**Verification**:
```bash
go test ./...
go vet ./...
```
```

## Best Practices

### Task Sizing
- ✅ 1-5 iterations per task
- ❌ Not "build entire app"
- ❌ Not "rename variable"

### Completion Criteria
- ✅ "API returns 200 with user object"
- ❌ "Make API better"

### Progress Updates
- Write for a stranger
- Include key decisions
- Note file locations
- Keep it concise

### Safety
- Start with max 10 iterations
- Monitor costs
- Run HITL (human-in-loop) first
- Keep CI green

## Stopping the Loop

### Automatic Stop
Coordinator outputs `<promise>COMPLETE</promise>` when:
- All PRD tasks marked done
- All verification checks pass

### Manual Stop
- Close VSCode chat
- Or delete PROGRESS.md to reset

## Troubleshooting

### Agents not appearing
- Restart VSCode
- Check files in `.github/agents/`
- Verify `.agent.md` extension

### Infinite loop
- Check PRD has clear completion criteria
- Verify PROGRESS.md is being updated
- Set max iterations in PRD

### Tests keep failing
- Executor will retry with fixes
- Check PROGRESS.md for error details
- May need human intervention

### Context confusion
- That's the point! Fresh context each iteration
- All memory in PROGRESS.md and git
- Trust the files, not conversation

## Customization

### Add Your Own Agent

Create `.github/agents/reviewer.agent.md`:

```markdown
---
name: Reviewer
description: Code review agent
tools: ['read', 'search']
handoffs:
  - label: Fix Issues
    agent: executor
    prompt: Fix the issues found in code review
---

# Code Reviewer
[Your instructions here]
```

### Modify Handoff Behavior

In any agent file:

```yaml
handoffs:
  - label: Custom Action
    agent: target-agent
    prompt: Detailed instructions
    send: true  # Auto-submit (true) or manual (false)
```

## Cost Estimation

Rough estimates per iteration:
- GPT-4: $0.30 - $1.00
- Claude Opus: $0.50 - $1.50
- Claude Sonnet: $0.15 - $0.50

20 iterations ≈ $10-30 depending on model and task complexity

## Credits

Based on:
- **Ralph Wiggum** pattern by Geoffrey Huntley
- **VSCode Custom Agents** (VSCode 1.106+)
- **Handoffs** feature for guided workflows

## License

MIT - Use freely, adapt as needed

## Links

- [VSCode Custom Agents Docs](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- [Original Ralph Pattern](https://github.com/snarktank/ralph)
- [Ralph Loop Examples](https://www.aihero.dev/getting-started-with-ralph)
