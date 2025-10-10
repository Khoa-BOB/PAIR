# PAIR

**Platform Architecture Iterative Refinement**

PAIR is a two-agent framework for designing software through iterative feature refinement. One agent (Adder) proposes new features, while the other (Remover) trims unnecessary ones. Together, they iteratively refine features to evolve requirements, balance scope, and shape software design.

## Overview

PAIR implements a dialectical approach to software design where two specialized agents work in opposition to create balanced, well-scoped feature sets:

- **Adder Agent (feature-architect)**: Analyzes project requirements and proposes innovative features that balance user experience, technical feasibility, and business value
- **Remover Agent (feature-remover)**: Critically evaluates existing features to identify low-value additions, reduce scope creep, and maintain strategic alignment

Through iterative cycles of addition and removal, the framework helps you:
- **Evolve requirements** from initial ideas to concrete feature sets
- **Balance scope** by preventing both feature bloat and under-specification
- **Shape design** through justified, documented decision-making
- **Maintain focus** on high-value features that serve end users

## Key Features

- **Structured Requirements Gathering**: 5-question process to capture project essentials
- **Schema-Driven Output**: All artifacts follow validated YAML schemas
- **Iterative Refinement**: Cycles of addition/removal rather than one-time definition
- **Justified Decisions**: Every feature change includes concrete reasoning
- **Status Tracking**: Complete history of additions and removals by round
- **Integration with Claude Code**: Built as custom slash commands and agents

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

```
.
├── .claude/
│   ├── agents/
│   │   ├── adder/              # Feature-architect agent definition
│   │   └── remover/            # Feature-remover agent definition
│   └── commands/               # Slash commands (describe_goal, add_features, remove_features, iterate)
├── config/
│   └── schemas/
│       ├── brief.yaml          # Project brief schema
│       └── feature.yaml        # Feature schema
└── registry/                   # Generated output (gitignored, created during execution)
    ├── brief.yaml              # Project description and requirements
    └── project-features.yaml   # All features with add/remove history
```

## Getting Started

### Prerequisites

- [Claude Code CLI](https://docs.claude.com/claude-code) installed and configured
- Git repository initialized in your project directory

### Workflow

#### 1. Create Project Brief

Start by describing your project through a guided requirements gathering process:

```bash
/describe_goal
```

This command will ask you 5 questions:
1. What is the main pain point you hope this software will address?
2. Who will use this software, and what are their main needs or challenges?
3. What are the most important features you need right away, and what can wait until later?
4. What budget, timeline, or compliance requirements should we keep in mind?
5. Do you have preferences for platforms, technology, or integrations with existing systems?

**Output**: `registry/brief.yaml` containing:
- Project name and problem description
- End user types
- Development methodology (Agile, Scrum, Kanban, etc.)
- User stories with priorities
- Technology preferences
- Hard constraints

#### 2. Generate Initial Features

Add your first set of features:

```bash
/add_features
```

The Adder agent will:
- Read your project brief from `registry/brief.yaml`
- Analyze requirements and user needs
- Generate 3 new features with scores (0-5), status, and justifications
- Append to `registry/project-features.yaml`

Each feature includes:
- Unique ID (F001, F002, etc.)
- Name and description
- Score indicating importance
- Status (true = active, false = removed)
- Round added and explanation

#### 3. Refine Features

Remove unnecessary or low-value features:

```bash
/remove_features
```

The Remover agent will:
- Read project brief and current features
- Evaluate each feature critically
- Remove at most 2 features per iteration
- Update `registry/project-features.yaml` with removal explanations

Removed features:
- Status changes to `false`
- Retain all metadata for historical tracking
- Include round removed and justification

#### 4. Iterate (Recommended)

**Option A: Automated Iteration**

Run multiple rounds automatically:

```bash
/iterate 3
```

This command will:
- Execute 3 rounds of add (3 features) + remove (up to 2 features)
- Display progress after each round
- Generate final statistics (total, active, removed, retention rate)

**Parameters**:
- `number_of_rounds` (optional): Number of cycles to run (default: 3, max: 10)
- Round numbers represent **iteration cycles** (1, 2, 3...), not cumulative counters

**Option B: Manual Iteration**

Alternate between `/add_features` and `/remove_features` for more control:

```bash
/add_features      # Round 2: Add 3 more features
/remove_features   # Round 2: Remove up to 2 features
/add_features      # Round 3: Add 3 more features
/remove_features   # Round 3: Remove up to 2 features
```

You can also specify round numbers explicitly:

```bash
/add_features 5      # Adds features with adding.round: 5
/remove_features 5   # Removes features with removing.round: 5
```

Or let the commands auto-calculate the next round based on existing features:

```bash
/add_features        # Calculates max_adding_round + 1
/remove_features     # Calculates max_removing_round + 1
```

## Data Schemas

### Brief Schema

```yaml
project_name: "Your Project Name"
problem_description: "The problem being solved"

end_users:
  - "User type 1"
  - "User type 2"

methodology: "Scrum"  # Options: Agile, Waterfall, Scrum, Kanban, Lean

user_stories:
  - id: "story.001"
    role: "Developer"
    goal: "Track code changes"
    benefit: "Maintain project history"
    priority: 1  # 1 = highest, 5 = lowest

technologies:
  - "React"
  - "Node.js"

hard_constraints:
  - "Must comply with GDPR"
  - "Budget under $50k"
```

### Feature Schema

```yaml
features:
  - feature_id: "F001"
    feature_name: "User Authentication"
    feature_description: "Secure login system with OAuth support"
    score: 4.5  # 0-5 float, importance rating
    status: true  # true = active, false = removed
    adding:
      round: 1
      explaination: "Core security requirement for user data protection aligning with story.001 for secure user access"
    category: "Security"  # Optional
    version: "1.0.0"      # Optional, semantic versioning
    owner: "Backend Team"  # Optional

  - feature_id: "F002"
    feature_name: "Dark Mode"
    feature_description: "Toggle between light and dark themes"
    score: 2.0
    status: false  # Removed
    adding:
      round: 1
      explaination: "Enhance user experience with theme options for better accessibility"
    removing:
      round: 2
      explaination: "Low priority compared to core features; represents scope creep given budget and timeline constraints. Can be added post-MVP after core functionality is proven."
```

## Agent Behaviors

### Adder Agent (feature-architect)

**Located**: `.claude/agents/adder/feature-architect.md`
**Model**: sonnet
**Invoked via**: Task tool with `subagent_type: "feature-architect"`

**Philosophy**: Propose innovative features that create user value while remaining technically feasible.

**Approach**:
- Systematic requirements analysis from project brief
- Analyzes end user needs and pain points
- Considers competitive differentiation and business value
- Proposes features in three tiers:
  - **Core essentials**: Must-have functionality for MVP
  - **Competitive differentiators**: Features that set the product apart
  - **Future innovations**: Forward-looking capabilities
- Balances user experience, technical feasibility, and business value
- Balances ambition with practical constraints

**Output**: Exactly 3 features per invocation in YAML format, directly appendable to project-features.yaml

### Remover Agent (feature-remover)

**Located**: `.claude/agents/remover/feature-remover.md`
**Model**: sonnet
**Invoked via**: Task tool with `subagent_type: "feature-remover"`

**Philosophy**: Apply critical evaluation to maintain focus on high-value features using the 80/20 rule.

**Approach**:
- Critical value assessment of all active features (status: true)
- Evaluates: alignment with requirements, redundancy, resource impact, technical debt, strategic fit
- Considers resource impact and technical debt
- Assesses strategic alignment with project goals
- Identifies scope creep and feature bloat
- Provides clear removal justifications with risk assessment

**Output**: JSON format with 0-2 removal recommendations per invocation:
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

## Key Principles

1. **Iterative Refinement**: Cycles of addition and removal rather than one-time feature definition. Each iteration adds 3 features and removes up to 2, creating a dialectical process that refines scope.
2. **Iteration-Based Round Numbering**: Round numbers represent **iteration cycles**, not global counters. Each `/iterate` run uses rounds 1, 2, 3... regardless of previous iterations. Manual commands auto-calculate next sequential round.
3. **Agent Invocation Architecture**: Commands invoke agents via Task tool, maintaining separation. Commands handle file I/O and orchestration; agents handle analysis and decision-making.
4. **Justified Changes**: Every feature addition/removal includes concrete reasoning (3-5 sentences) that references specific user stories, constraints, and strategic value.
5. **Status Tracking**: Features are never deleted; status is updated to false when removed. This maintains complete historical record with both adding and removing explanations.
6. **Schema Compliance**: All outputs conform to validated YAML schemas. Agents are provided with schema specifications to ensure compliance.
7. **Balanced Perspectives**: Opposing agents create tension that drives better decisions

## Example Workflow

### Quick Start (Recommended)

```bash
# Initialize project
/describe_goal
# Answer 5 questions about your project...

# Run automated iteration
/iterate 3
# Executes 3 rounds of add/remove cycles
# Final result: ~5 active features after refinement
```

### Manual Workflow

```bash
# Initialize project
/describe_goal
# Answer: Building a Python learning platform...

# Round 1
/add_features
# Output: 3 features added (F001-F003) - Core essentials

/remove_features
# Output: 0 features removed (all are essential)

# Round 2
/add_features
# Output: 3 more features added (F004-F006) - Enhancements

/remove_features
# Output: 2 features removed (F005, F006 marked as status: false)

# Round 3
/add_features
# Output: 3 more features added (F007-F009) - Advanced features

/remove_features
# Output: 2 features removed (F008, F009 marked as status: false)

# Review results
cat registry/project-features.yaml
# See all 9 features with complete add/remove history
# Active: F001, F002, F003, F004, F007 (5 features)
# Removed: F005, F006, F008, F009 (4 features)
```

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

## Important Notes

- **Always run `/describe_goal` first** to create the project brief before generating features
- **Round numbers are iteration-based**, not cumulative across multiple `/iterate` runs
- **Agents return different formats**: feature-architect returns YAML, feature-remover returns JSON
- **Status field is boolean**: `true` = active, `false` = removed (not strings)
- **Feature IDs must be unique** and follow pattern `F###` or `F####` (e.g., F001, F1234)
- **Scores are floats 0-5**, representing feature importance/priority
- **User story priorities are 1-5**, where 1 is highest priority (most important)
- **Registry folder is gitignored** by default - outputs are generated artifacts, not source code

## Use Cases

- **Startup MVPs**: Rapidly iterate on product scope before development begins
- **Enterprise Projects**: Document and justify feature decisions for stakeholders
- **Open Source Planning**: Collaboratively refine roadmaps with community input
- **Academic Projects**: Apply structured decision-making to software design courses
- **Consulting**: Help clients articulate and prioritize their needs

## Contributing

This is a framework built on Claude Code's agent and command system. To extend or customize:

1. **Modify agents**: Edit `.claude/agents/adder/feature-architect.md` or `.claude/agents/remover/feature-remover.md`
2. **Adjust schemas**: Update `config/schemas/brief.yaml` or `config/schemas/feature.yaml`
3. **Add commands**: Create new slash commands in `.claude/commands/`
4. **Customize workflows**: Modify existing command prompts to fit your process

## License

[Add your license here]

## Support

For issues or questions about Claude Code, visit [https://docs.claude.com/claude-code](https://docs.claude.com/claude-code)

For issues specific to PAIR, please open an issue in this repository.
