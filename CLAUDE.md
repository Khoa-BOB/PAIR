# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**PAIR** (Platform Architecture Iterative Refinement) is a two-agent framework for designing software through iterative feature refinement. The system uses:
- **Adder Agent** (feature-architect): Proposes new features based on requirements
- **Remover Agent** (feature-remover): Evaluates and removes unnecessary features

The framework helps evolve software requirements, balance scope, and shape design through iterative cycles.

## Architecture

### Two-Agent System with Task Tool Invocation

The framework uses Claude Code's Task tool to invoke specialized agents, maintaining clear separation between command orchestration and agent reasoning:

**Command Layer** (`.claude/commands/`):
- Handles file I/O, context gathering, and result processing
- Invokes agents via Task tool with `subagent_type` parameter
- Manages round numbering and workflow orchestration

**Agent Layer** (`.claude/agents/`):
- **feature-architect**: Analyzes requirements and generates features (returns YAML)
- **feature-remover**: Evaluates features for removal (returns JSON)
- Agents do the thinking; commands do the I/O

### Directory Structure
- `.claude/agents/adder/` - feature-architect agent definition
- `.claude/agents/remover/` - feature-remover agent definition
- `.claude/commands/` - Slash commands (describe_goal, add_features, remove_features, iterate)
- `config/schemas/` - YAML schemas (brief.yaml, feature.yaml)
- `registry/` - Generated outputs (brief.yaml, project-features.yaml)

### Core Workflows

#### 1. `/describe_goal` - Project Initialization
- Interactive 5-question requirements gathering
- Generates `registry/brief.yaml` with:
  - Project name and problem description
  - End users and methodology
  - User stories with priorities (1-5, where 1 is highest)
  - Technology preferences
  - Hard constraints
- Uses schema from `config/schemas/brief.yaml`

#### 2. `/add_features [round_number]` - Feature Generation
**Command signature**: `/add_features [round_number]`

**Parameters**:
- `round_number` (optional): Iteration round to use for `adding.round` field
  - If provided: Uses the specified round (from `/iterate`)
  - If omitted: Calculates `max_adding_round + 1` from existing features
  - Fallback: Round 1 if no features exist

**Workflow**:
1. Initialize `registry/project-features.yaml` if needed
2. Read context: brief.yaml, project-features.yaml, feature.yaml schema
3. Determine round number (parameter > auto-calculate > fallback)
4. **Invoke Task tool** with `subagent_type: "feature-architect"`
5. Agent analyzes requirements and generates 3 features in YAML format
6. Append agent output to project-features.yaml

**Agent Prompt Includes**:
- Full project brief content
- Existing active features summary
- Feature schema specification
- Current round number
- Instructions to generate exactly 3 features with:
  - Sequential feature IDs (F001, F002, etc.)
  - Scores (0-5 float)
  - Status: true
  - Adding section with round and explanation (3-5 sentences)

#### 3. `/remove_features [round_number]` - Feature Evaluation
**Command signature**: `/remove_features [round_number]`

**Parameters**:
- `round_number` (optional): Iteration round to use for `removing.round` field
  - If provided: Uses the specified round (from `/iterate`)
  - If omitted: Calculates `max_removing_round + 1` from existing removals
  - Fallback: Round 1 if no removals exist

**Workflow**:
1. Read context: brief.yaml, project-features.yaml
2. Determine round number (parameter > auto-calculate > fallback)
3. **Invoke Task tool** with `subagent_type: "feature-remover"`
4. Agent evaluates all active features (status: true)
5. Agent returns JSON with up to 2 removal recommendations
6. For each removal:
   - Change status: true → false
   - Add removing section with round and explanation

**Agent Prompt Includes**:
- Full project brief content
- All current features (active and removed)
- Current round number
- Instructions to return JSON format:
  ```json
  {
    "removals": [
      {
        "feature_id": "F###",
        "explanation": "Detailed removal justification..."
      }
    ]
  }
  ```

#### 4. `/iterate [number_of_rounds]` - Automated Iteration
**Command signature**: `/iterate [number_of_rounds]`

**Parameters**:
- `number_of_rounds` (optional): Number of add/remove cycles (default: 3, max: 10)

**Workflow**:
For each round from 1 to N:
1. Display round start message
2. Invoke `/add_features [X]` (passes current round number)
3. Display added features summary
4. Invoke `/remove_features [X]` (passes current round number)
5. Display removed features summary
6. Display round complete message

After all rounds:
- Generate statistics (total, active, removed, retention rate)
- Display summary and file path

**Key Behavior**: Round numbers represent **iteration cycles**, not global counters. Each `/iterate` run uses rounds 1, 2, 3, ... regardless of previous iterations.

### Data Schemas

#### Brief Schema (`config/schemas/brief.yaml`)
```yaml
project_name: string
problem_description: string
end_users: array of strings
methodology: "Agile" | "Waterfall" | "Scrum" | "Kanban" | "Lean"
user_stories:
  - id: string
    role: string
    goal: string
    benefit: string
    priority: integer (1-5, where 1 is highest)
technologies: array of strings
hard_constraints: array of strings
```

#### Feature Schema (`config/schemas/feature.yaml`)
```yaml
features:
  - feature_id: string (pattern: ^F\d{3,4}$)
    feature_name: string
    feature_description: string
    score: float (0-5)
    status: boolean (true = active, false = removed)
    adding:
      round: integer
      explaination: string
    removing: (optional, added when removed)
      round: integer
      explaination: string
    category: string (optional)
    version: string (optional, pattern: ^[0-9]+\.[0-9]+\.[0-9]+$)
    owner: string (optional)
```

### Agent Behaviors

#### Adder Agent (feature-architect)
**Located**: `.claude/agents/adder/feature-architect.md`
**Model**: sonnet
**Invoked via**: `Task tool` with `subagent_type: "feature-architect"`

**Approach**:
- Systematic requirements analysis
- Tiered feature generation: core essentials, competitive differentiators, future innovations
- Balances user experience, technical feasibility, and business value
- Returns: YAML format, exactly 3 features per invocation

**Output Format**: YAML features array directly appendable to project-features.yaml

#### Remover Agent (feature-remover)
**Located**: `.claude/agents/remover/feature-remover.md`
**Model**: sonnet
**Invoked via**: `Task tool` with `subagent_type: "feature-remover"`

**Approach**:
- Critical value assessment using 80/20 rule
- Evaluates: alignment with requirements, redundancy, resource impact, technical debt, strategic fit
- Identifies scope creep and feature bloat
- Returns: JSON with 0-2 removal recommendations

**Output Format**: JSON with removals array containing feature_id and explanation

## Common Commands

### Starting a New Project
```bash
# 1. Create project brief
/describe_goal
# Answer 5 questions about your project

# 2. Run automated iteration (recommended)
/iterate 3
# Executes 3 rounds of add (3 features) + remove (up to 2 features)

# OR: Manual iteration
/add_features      # Round 1: Add 3 features
/remove_features   # Round 1: Remove up to 2 features
/add_features      # Round 2: Add 3 features
/remove_features   # Round 2: Remove up to 2 features
```

### Continuing an Existing Project
```bash
# Run additional iterations
/iterate 2
# Rounds 1-2 (iteration-based, not continuation of previous rounds)

# Add features manually with specific round
/add_features 5
# Adds 3 features with adding.round: 5

# Add features with auto-calculated round
/add_features
# Calculates next round from existing features
```

### Working with the Registry
```bash
# View project brief
cat registry/brief.yaml

# View all features with history
cat registry/project-features.yaml

# Count active features
grep "status: true" registry/project-features.yaml | wc -l

# Count removed features
grep "status: false" registry/project-features.yaml | wc -l
```

## Key Principles

### 1. Iterative Refinement
Cycles of addition and removal rather than one-time feature definition. Each iteration adds 3 features and removes up to 2, creating a dialectical process that refines scope.

### 2. Iteration-Based Round Numbering
Round numbers represent **iteration cycles**, not global counters:
- First `/iterate 3`: rounds 1, 2, 3
- Second `/iterate 3`: rounds 1, 2, 3 (resets)
- Manual `/add_features`: auto-calculates next sequential round

### 3. Agent Invocation Architecture
Commands invoke agents via Task tool, maintaining separation:
- **Commands**: File I/O, context gathering, workflow orchestration
- **Agents**: Analysis, generation, evaluation, decision-making
- **Never** bypass agents by generating features directly in commands

### 4. Justified Changes
Every feature addition/removal includes concrete reasoning (3-5 sentences) that references:
- Specific user stories
- Project constraints
- Alignment with requirements
- Strategic value assessment

### 5. Status Tracking
Features are never deleted; status is updated to false when removed. This maintains complete historical record of all decisions with both adding and removing explanations.

### 6. Schema Compliance
All outputs must conform to defined YAML schemas. Agents are provided with schema specifications in their prompts to ensure compliance.

## Agent Invocation Examples

### Invoking feature-architect:
```
Task tool:
  subagent_type: "feature-architect"
  prompt: |
    PROJECT BRIEF: [full brief.yaml content]
    EXISTING FEATURES: [summary of active features]
    FEATURE SCHEMA: [schema specification]
    CURRENT ROUND: [round number]

    Generate exactly 3 features following the schema...
```

### Invoking feature-remover:
```
Task tool:
  subagent_type: "feature-remover"
  prompt: |
    PROJECT BRIEF: [full brief.yaml content]
    CURRENT FEATURES: [all features including removed]
    CURRENT ROUND: [round number]

    Evaluate ALL active features and identify up to 2 for removal...
    Return JSON: {"removals": [{"feature_id": "F###", "explanation": "..."}]}
```

## Important Notes

- **Always run `/describe_goal` first** to create the project brief before generating features
- **Round numbers are iteration-based**, not cumulative across multiple `/iterate` runs
- **Agents return different formats**: feature-architect returns YAML, feature-remover returns JSON
- **Status field is boolean**: true = active, false = removed (not strings)
- **Feature IDs must be unique** and follow pattern F### or F#### (e.g., F001, F1234)
- **Scores are floats 0-5**, representing feature importance/priority
- **User story priorities are 1-5**, where 1 is highest priority
