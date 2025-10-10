---
name: remove_features
description: Remove features for the software.
---

**Command:** `/remove_features [round_number]`

**Purpose:** Invoke the feature-remover subagent to evaluate existing features and identify up to 2 that should be removed from the project.

## Parameters:
- `round_number` (optional): The iteration round number to use for the `removing.round` field. If not provided, will calculate based on existing removals.

## Execution Steps:

1. **Gather Context**: Read the following files to provide context to the agent:
   - `registry/brief.yaml` (project requirements and constraints)
   - `registry/project-features.yaml` (all features including active and removed)

2. **Determine Round Number**:
   - If `round_number` parameter is provided, use it
   - Otherwise, find the highest `removing.round` value in existing features and add 1
   - Fallback: If no removals exist, use round 1
   - This will be used in the `removing.round` field

3. **Invoke Feature-Remover Agent**: Use the Task tool to invoke the specialized agent:
   - **Tool**: Task
   - **subagent_type**: "feature-remover"
   - **Prompt structure**:
     ```
     You are the feature-remover agent for the PAIR framework, evaluating features for potential removal.

     PROJECT BRIEF:
     [Full content from registry/brief.yaml]

     CURRENT FEATURES:
     [Full content from registry/project-features.yaml]

     CURRENT ROUND: [calculated_round_number]

     TASK: Evaluate ALL features that currently have status: true and identify up to 2 features that should be removed because they:
     - Don't align with project requirements or constraints
     - Are redundant with other active features
     - Have low value relative to implementation complexity
     - Represent scope creep or feature bloat
     - Conflict with hard constraints or user priorities

     For each feature you recommend removing, provide:
     1. feature_id (exact ID from the features file)
     2. Concrete explanation (3-5 sentences) of why it should be removed
     3. Specific evidence from the brief or feature set supporting your decision

     Return your response as JSON in this exact format:
     {
       "removals": [
         {
           "feature_id": "F###",
           "explanation": "Detailed removal justification here..."
         }
       ]
     }

     If no features should be removed (all active features are valuable), return:
     {
       "removals": []
     }

     Remember: Remove AT MOST 2 features. Be decisive but strategic.
     ```

4. **Process Agent Output**:
   - Receive JSON response from the agent
   - Parse the removals array
   - For each removal recommendation:
     a. Locate the feature in `registry/project-features.yaml`
     b. Change `status: true` to `status: false`
     c. Add `removing:` section with:
        - `round: [calculated_round_number]`
        - `explaination: [agent's explanation]`
     d. Update the YAML file

5. **Confirmation**: Display summary message: "Evaluated features and removed [N] in round [X]" where N is 0-2