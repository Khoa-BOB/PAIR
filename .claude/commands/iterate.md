---
name: iterate
description: Run multiple rounds of feature addition and removal iterations
---

**Command:** `/iterate [number_of_rounds]`

**Purpose:** Execute multiple iterative cycles of the Adder and Remover agents to refine the feature set through successive rounds of addition and removal.

## Parameters:
- `number_of_rounds` (optional): Number of add/remove cycles to run. Default is 3 if not specified.

## Execution Steps:

### 1. Validation
- Check if `registry/brief.yaml` exists. If not, prompt user to run `/describe_goal` first.
- Parse the number of rounds from user input (default to 3 if not provided)
- Validate that number_of_rounds is between 1 and 10

### 2. Initialize
- If `registry/project-features.yaml` doesn't exist, create it with `touch registry/project-features.yaml`
- Display starting message: "Starting [N] rounds of feature iteration..."

### 3. Execute Iteration Cycles

For each round from 1 to N:

#### Round Start
- Display: "=== Round [X] of [N] ==="

#### Add Phase
- Display: "Adding features (Round [X])..."
- Use the SlashCommand tool to invoke `/add_features [X]` (pass the round number as argument)
- Wait for the add_features command to complete
- Display summary: "Added 3 new features in round [X]"

#### Remove Phase
- Display: "Evaluating features for removal (Round [X])..."
- Use the SlashCommand tool to invoke `/remove_features [X]` (pass the round number as argument)
- Wait for the remove_features command to complete
- Display summary: "Evaluated features and removed up to 2 in round [X]"

#### Round Complete
- Display: "Round [X] complete"
- Add a separator line

### 4. Final Summary

After all rounds complete:
- Display: "=== Iteration Complete: [N] rounds finished ==="
- Read `registry/project-features.yaml` to generate statistics:
  - Total features generated across all rounds
  - Total features currently active (status: true)
  - Total features removed (status: false)
  - Feature retention rate (active/total)
- Display the summary statistics
- Provide path to view results: "View all features: registry/project-features.yaml"

## Example Usage:

```bash
# Run 3 rounds (default)
/iterate

# Run 5 rounds
/iterate 5

# Run single round
/iterate 1
```

## Notes:
- Each iteration adds 3 features and removes at most 2 features
- Features are never deleted, only marked with status: false
- All changes include round numbers and justifications
- Progress is displayed after each phase for transparency
