---
name: add_features
description: Generate features for the software.
---

**Command:** `/add_features [round_number]`

**Purpose:** Invoke the feature-architect subagent to analyze the software description and generate 3 valuable features for the project.

## Parameters:
- `round_number` (optional): The iteration round number to use for the `adding.round` field. If not provided, will calculate based on existing features.

## Execution Steps:

1. **Init file**: If there is no file `registry/project-features.yaml`, create it with `touch registry/project-features.yaml`

2. **Gather Context**: Read the following files to provide context to the agent:
   - `registry/brief.yaml` (project requirements and constraints)
   - `registry/project-features.yaml` (existing features, if any)
   - `config/schemas/feature.yaml` (feature schema specification)

3. **Determine Round Number**:
   - If `round_number` parameter is provided, use it
   - Otherwise, find the highest `adding.round` value in existing features and add 1
   - Fallback: If no features exist, use round 1
   - This will be used in the `adding.round` field

4. **Invoke Feature-Architect Agent**: Use the Task tool to invoke the specialized agent:
   - **Tool**: Task
   - **subagent_type**: "feature-architect"
   - **Prompt structure**:
     ```
     You are the feature-architect agent for the PAIR framework, generating features for a software project.

     PROJECT BRIEF:
     [Full content from registry/brief.yaml]

     EXISTING FEATURES:
     [Full content from registry/project-features.yaml, or "No features yet" if empty]

     FEATURE SCHEMA:
     [Full content from config/schemas/feature.yaml]

     CURRENT ROUND: [calculated_round_number]

     TASK: Generate exactly 3 new features that:
     1. Follow the feature schema precisely (feature_id, feature_name, feature_description, score, status, adding section)
     2. Address user stories and constraints from the brief
     3. Complement existing active features without duplication
     4. Include concrete, detailed explanations (3-5 sentences) for why each feature should be added
     5. Use sequential feature IDs starting from F[next_number]
     6. Set status: true for all new features
     7. Set adding.round to [calculated_round_number]
     8. Assign appropriate scores (0-5 float) based on importance and alignment

     Return ONLY valid YAML formatted as a features array that can be directly appended to the project-features.yaml file.
     Start with a newline, then the feature entries with proper indentation (2 spaces).
     ```

5. **Process Agent Output**:
   - Receive YAML-formatted features from the agent
   - Validate that output contains exactly 3 features
   - Verify format matches the schema
   - Append agent's output to `registry/project-features.yaml`

6. **Confirmation**: Display summary message: "Added 3 new features in round [X]"