---
name: describe_goal
description: Create structured problem brief from user input for ideation workflows
---

**Command:** `/describe_goal`

**Purpose:** Transform user problem description into structured briefs that enable effective ideation across all domains.

**Implementation Steps:**

```bash

# 1. Create output directory
mkdir -p registry

# 3. Read configs/schemas/brief.yaml 


# 2. Generate complete brief of the software
claude << EOF > registry/brief.yaml
Generate a structured YAML brief for this software.


**REQUIREMENTS GATHERING**
I'll guide you through a step-by-step requirements gathering process. Each time you answer a question, I'll acknowledge your response and ask the next question.

Question 1: What is the main pain point you hope this software will address?
Question 2: Who will use this software, and what are their main needs or challenges?
Question 3: What are the most important features you need right away, and what can wait until later?
Question 4: What budget, timeline, or compliance requirements should we keep in mind?
Question 5: Do you have preferences for platforms, technology, or integrations with existing systems?



**YAML STRUCTURE REQUIREMENTS:**

project_name: "[Name of the project]"
problem_description: "[Short description of the problem being addressed]"

end_users:
  - "[User type 1]"
  - "[User type 2]"

methodology: "Scrum"   # Agile | Waterfall | Scrum | Kanban | Lean

features:
  - "[Feature 1]"
  - "[Feature 2]"
  - "[Feature 3]"

user_stories:
  - id: "story.001"
    role: "[Actor, e.g., developer, student, admin]"
    goal: "[Goal the user wants to achieve]"
    benefit: "[Reason for the goal (value/benefit)]"
    priority: 1   # 1 = highest, 5 = lowest

  - id: "story.002"
    role: "[Actor]"
    goal: "[Goal]"
    benefit: "[Reason/benefit]"
    priority: 2

technologies:
  - "[Tech 1]"
  - "[Tech 2]"

hard_constraints:
  - "[Constraint ensuring relevance]"
  - "[Constraint ensuring feasibility]"
  - "[Constraint ensuring domain alignment]"


**GUIDELINES:**
- Gather all the information by going over 5 questions from the REQUIREMENTS GATHERING
- Problem description: Get the problem the software want to solve from the response of question 1
- End users: Analyze the end users from the response of question 2
- FEATURES: Gather desired features from the response of question 3
- Constraints: Analyze the limitation from the response of question 4
- Technologies: Analyze the desired techonologies from the response of question 5

**OUTPUT:** Generate only valid YAML with no additional text, comments, or explanations.
EOF
```
