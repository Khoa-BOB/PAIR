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

```
.
├── .claude/
│   ├── agents/
│   │   ├── adder/              # Feature-architect agent
│   │   └── remover/            # Feature-remover agent
│   └── commands/               # Slash commands for workflows
├── config/
│   └── schemas/
│       ├── brief.yaml          # Project brief schema
│       └── feature.yaml        # Feature schema
└── registry/                   # Generated output (created during execution)
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

#### 4. Iterate

Alternate between `/add_features` and `/remove_features` to refine your feature set through multiple rounds:

```bash
/add_features      # Round 2: Add 3 more features
/remove_features   # Round 2: Remove up to 2 features
/add_features      # Round 3: Add 3 more features
/remove_features   # Round 3: Remove up to 2 features
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
    score: 4.5  # 0-5, importance rating
    status: true  # true = active, false = removed
    round_added: 1
    added_explanation: "Core security requirement for user data protection"
    category: "Security"  # Optional
    version: "1.0.0"      # Optional, semantic versioning
    owner: "Backend Team"  # Optional

  - feature_id: "F002"
    feature_name: "Dark Mode"
    feature_description: "Toggle between light and dark themes"
    score: 2.0
    status: false  # Removed
    round_added: 1
    added_explanation: "Enhance user experience with theme options"
    round_removed: 2
    removed_explanation: "Low priority compared to core features; can be added post-MVP"
```

## Agent Behaviors

### Adder Agent (feature-architect)

**Philosophy**: Propose innovative features that create user value while remaining technically feasible.

**Approach**:
- Analyzes end user needs and pain points from the brief
- Considers competitive differentiation and business value
- Proposes features in three tiers:
  - **Core essentials**: Must-have functionality for MVP
  - **Competitive differentiators**: Features that set the product apart
  - **Future innovations**: Forward-looking capabilities
- Balances ambition with practical constraints

**Output**: 3 features per round with concrete justifications

### Remover Agent (feature-remover)

**Philosophy**: Apply critical evaluation to maintain focus on high-value features.

**Approach**:
- Evaluates features against the 80/20 rule (Pareto principle)
- Considers resource impact and technical debt
- Assesses strategic alignment with project goals
- Identifies scope creep and redundancy
- Provides clear removal justifications with risk assessment

**Output**: At most 2 removals per round with detailed explanations

## Key Principles

1. **Iterative Refinement**: Cycles of addition and removal rather than one-time feature definition
2. **Justified Changes**: Every feature addition/removal must include concrete reasoning
3. **Status Tracking**: Features are never deleted; their status is updated when removed
4. **Round Tracking**: Each operation records the iteration round for historical context
5. **Schema Compliance**: All outputs conform to validated YAML schemas
6. **Balanced Perspectives**: Opposing agents create tension that drives better decisions

## Example Workflow

```bash
# Initialize project
/describe_goal
# Answer: Building a task management app for remote teams...

# Round 1
/add_features
# Output: 3 features added (F001-F003)

/remove_features
# Output: 1 feature removed (F002 marked as status: false)

# Round 2
/add_features
# Output: 3 more features added (F004-F006)

/remove_features
# Output: 2 features removed (F001, F005 marked as status: false)

# Review results
cat registry/project-features.yaml
# See all features with complete add/remove history
```

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
