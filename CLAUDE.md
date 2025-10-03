# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**PAIR** (Platform Architecture Iterative Refinement) is a two-agent framework for designing software through iterative feature refinement. The system uses:
- **Adder Agent** (feature-architect): Proposes new features based on requirements
- **Remover Agent** (feature-remover): Evaluates and removes unnecessary features

The framework helps evolve software requirements, balance scope, and shape design through iterative cycles.

## Architecture

### Directory Structure
- `.claude/agents/adder/` - Contains the Adder agent definition (feature-architect)
- `.claude/agents/remover/` - Contains the Remover agent definition (feature-remover)
- `.claude/commands/` - Custom slash commands for the workflow
- `config/schemas/` - YAML schemas for data validation
- `registry/` - Generated output files (created during workflow execution)

### Core Workflows

The system operates through three main commands:

1. **`/describe_goal`** - Initial project setup
   - Guides user through 5-question requirements gathering
   - Generates `registry/brief.yaml` with project description, users, features, user stories, technologies, and constraints
   - Uses schema defined in `config/schemas/brief.yaml`

2. **`/add_features`** - Feature generation (Adder agent)
   - Reads project description from `registry/brief.yaml`
   - Generates 3 new features per iteration using `config/schemas/feature.yaml`
   - Creates or appends to `registry/project-features.yaml`
   - Includes concrete explanations for why each feature is added

3. **`/remove_features`** - Feature evaluation (Remover agent)
   - Reads project brief and current features
   - Evaluates all features and removes at most 2 per iteration
   - Updates `registry/project-features.yaml` with removal explanations
   - Maintains feature status tracking

### Data Schemas

**Brief Schema** (`config/schemas/brief.yaml`):
- Required: project_name, problem_description, end_users
- Methodology options: Agile, Waterfall, Scrum, Kanban, Lean
- User stories with priority (1-5, where 1 is highest)

**Feature Schema** (`config/schemas/feature.yaml`):
- Required: feature_id (format: F### or F####), feature_name, feature_description, score, status
- Status: boolean (true = active, false = removed)
- Adding/removing metadata: round number and explanation
- Score: float from 0-5 indicating feature importance
- Optional: category, version (semantic), owner

### Agent Behaviors

**Adder Agent** (feature-architect):
- Analyzes requirements and proposes innovative features
- Balances user experience, technical feasibility, and business value
- Offers both incremental improvements and transformative features
- Uses tiered approach: core essentials, competitive differentiators, future innovations

**Remover Agent** (feature-remover):
- Critically evaluates feature value and alignment
- Applies 80/20 rule to identify low-value features
- Considers resource impact, technical debt, and strategic alignment
- Provides clear removal justifications with risk assessment

## Common Commands

### Starting a New Project
```bash
# 1. Run describe_goal to create project brief
/describe_goal
# Answer the 5 requirements questions
# Output: registry/brief.yaml

# 2. Generate initial features
/add_features
# Output: registry/project-features.yaml with 3 features

# 3. Refine features by removing unnecessary ones
/remove_features
# Updates: registry/project-features.yaml (marks up to 2 features as removed)

# 4. Iterate: alternate between /add_features and /remove_features
```

### Working with the Registry
The `registry/` directory is created during workflow execution and contains:
- `brief.yaml` - Project definition and requirements
- `project-features.yaml` - All features with add/remove history and status tracking

## Key Principles

1. **Iterative Refinement**: The framework emphasizes cycles of addition and removal rather than one-time feature definition
2. **Justified Changes**: Every feature addition/removal must include concrete reasoning
3. **Status Tracking**: Features are never deleted; their status is updated to false when removed
4. **Round Tracking**: Each add/remove operation records the iteration round for historical context
5. **Schema Compliance**: All outputs must conform to the defined YAML schemas
