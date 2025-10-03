---
name: remove_features
description: Remove features for the software.
---

**Command:** `/remove_features`

**Purpose:** Invoke feature-remover subagent to read the software description from brief.yaml and projects features from project_features.yaml

## Execution Steps:

1. **Project understanding:** Read software description from `registry/brief.yaml`
2. **Locate subagent:**  locate the subagent in `.claude/agents/remover`
3. **Features reviewing:** The agent reads the current features of the software from `registry/project_features.yaml`
3. **How to remove:**
    - Iterate over every features in the `registry/project_features.yaml`
    - Evaluate each feature to decide removing at most 2 features that are no longer needed for the software.
    - Adding the concrete explaination why the feature is removed.