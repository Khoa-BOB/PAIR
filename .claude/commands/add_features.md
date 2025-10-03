---
name: add_features
description: Generate features for the software.
---

**Command:** `/add_features`

**Purpose:** Invoke feature-architect subagent to read the software description from brief.yaml and software features if existing to generate more valuable features for the project. 

## Execution Steps:

1. **Init file** If there is no file `/registry/project-features.yaml`, run the bash command `touch project-features.yaml` to create initial file.
2. **Locate subagent:**  locate the subagent in `.claude/agents/adder`
3. **Project understanding:** Read software description from `registry/brief.yaml`
4. **Read the schema for a feature:** Read `.calude/feature.yaml`
5. **How to generate**
    - Following the provided schema to generate 3 more features that can meet the usecase of the software.
    - Concate them to the file `/registry/project-features.yaml` with the concrete explaination why the feature should be added.