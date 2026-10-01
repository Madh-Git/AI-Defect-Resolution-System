# Workflow Architecture

```text
User
 ↓

Agent 1
Jira Ticket Retrieval

 ↓

Agent 2
Bug Classification

 ↓

Agent 3
Historical Ticket Analysis

 ↓

Agent 4
Repository Analysis

 ↓

Agent 5
Root Cause Analysis

 ↓

Agent 6
Code Fix Recommendation

 ↓

Agent 7
Source Code Recommendation

 ↓

Agent 8
Engineering Report Generator

 ↓

Final Engineering Report
```

## Workflow Description

The workflow starts when a user enters a Jira Ticket ID.

The system:

1. Retrieves ticket details.
2. Classifies the defect.
3. Finds historical incidents.
4. Maps the defect to Medusa repository modules.
5. Identifies probable root causes.
6. Recommends engineering fixes.
7. Identifies likely source-code investigation targets.
8. Generates a final engineering defect resolution report.