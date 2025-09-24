# Terminal Session 1 - Claude Code Discussion & Learning

## Session Overview
Date: 2025-09-24
Focus: Spec Kit setup, CLI tools, and AI assistant configuration

---

## Key Discussions & Learnings

### 1. Spec Kit CLI Commands Issue

**Problem**: User tried to use `/constitution` command directly in terminal
- Command: `specify constitution "Create principles..."`
- **Error**: "No such command 'constitution'"

**Resolution**:
- Spec Kit only has `init` and `check` commands at CLI level
- `/constitution` and other slash commands are meant for use WITHIN AI development environments (Claude Code, Qwen, etc.)
- Correct workflow: `specify init` ’ open in AI tool ’ use `/constitution` within AI interface

### 2. AI CLI Tools Available
From `specify check` output:
-  Git version control
-  Claude Code CLI
-  Gemini CLI
-  Qwen Code CLI
-  Codex CLI
- L VS Code, Cursor IDE, Windsurf, etc. (not found)

### 3. Qwen Code Experience

**Authentication & Usage**:
- User successfully authenticated to Qwen
- Currently creating an application in another terminal
- No payment details provided - using free tier

**Free Tier Limits**:
- **2,000 requests per day** (no token limits)
- **60 requests per minute** rate limit
- Generous for development/testing
- Free tier intended for non-commercial use
- May train on user data (like other free AI services)

### 4. Gemini Pro License Integration

**Question**: Can Gemini Pro license be used with Gemini CLI?

**Answer**: Yes, multiple ways:
- **Direct Authentication**: Login with Google account (auto-detects Pro license)
- **API Key**: Set `GOOGLE_API_KEY` or `GEMINI_API_KEY` environment variables
- **Vertex AI**: Use `GOOGLE_GENAI_USE_VERTEXAI=true`

**Pro License Benefits**:
- Higher rate limits than free tier (60 req/min, 1000 req/day for free)
- Access to Gemini 2.5 Pro with 1M token context window
- Shared quotas between CLI and Code Assist
- Professional features for multiple agents

### 5. Qwen Code Automation Flags

**Question**: Qwen Code equivalent to Claude's `--dangerously-skip-permissions`?

**Key Options**:
```bash
# Most permissive (closest equivalent)
qwen --yolo                    # Auto-approve ALL tools

# Granular approval control
qwen --approval-mode yolo      # Auto-approve all
qwen --approval-mode auto_edit # Auto-approve only edits
qwen --approval-mode default   # Prompt for approval

# Additional safety/automation flags
qwen --all-files              # Include all files in context
qwen --checkpointing          # Enable file edit checkpoints
qwen --debug                  # Debug mode
qwen --allowed-tools tool1,tool2  # Whitelist specific tools
```

**Recommended for Spec Kit**:
```bash
# Most automated
qwen --yolo --all-files --checkpointing

# Balanced approach
qwen --approval-mode auto_edit --all-files --checkpointing
```

### 6. Single Responsibility Principle (SRP)

**Definition**: "A class should have only one reason to change"
- Each class/module/function should have one clear responsibility
- Part of SOLID principles in object-oriented design

**Benefits**:
- Easier to understand and test
- Better maintainability and reusability
- Lower coupling between components
- Changes affect only one area

**Example Context**: Found in user's Specify project constitution at `/SpecKitTinkering/.specify/memory/constitution.md`

---

## Technical Setup Notes

### Current Environment
- Working Directory: `/Users/mondweep/DxSure2/SpecKitTinkering`
- Git repo: Yes
- Platform: macOS (Darwin 24.5.0)
- Multiple AI CLI tools available

### File System Observations
- Spec Kit project structure exists under `SpecKitTinkering/`
- Constitution and specs already present in `.specify/memory/`
- Receipt parser API spec also found in memory

---

## Action Items & Next Steps

1. **Spec Kit Workflow**: Use AI tools (Qwen, Claude, Gemini) for slash commands, not terminal
2. **Qwen Usage**: Monitor free tier limits (2K req/day) during development
3. **Gemini Integration**: Configure API key if higher quotas needed
4. **Automation**: Use `--yolo` or `--approval-mode` flags for streamlined development

---

## CLI Command References

### Qwen Code
```bash
qwen                           # Start interactive mode
qwen --yolo                   # Auto-approve all actions
qwen --approval-mode auto_edit # Auto-approve edits only
qwen --all-files              # Include all files
qwen --checkpointing          # Enable checkpoints
```

### Spec Kit
```bash
specify init <project>        # Initialize new project
specify check                 # Check tool availability
```

### Gemini CLI
```bash
export GOOGLE_API_KEY="key"   # Set API key
gemini                        # Start interactive mode
```

---

*Session captured on 2025-09-24 via Claude Code*