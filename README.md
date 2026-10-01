# AI E-Commerce Defect Resolution System

A multi-agent engineering workflow built using Microsoft Copilot Studio that transforms a Jira ticket ID into a structured Engineering Defect Resolution Report.

The solution coordinates specialized AI agents for ticket retrieval, defect classification, historical incident analysis, repository analysis, root cause analysis, code-fix recommendations, source-code investigation guidance, and final report generation.

> **Project status:** Portfolio proof of concept  
> **Domain:** E-Commerce Engineering and Defect Management  
> **Reference repository:** Medusa open-source commerce platform  
> **Example input:** `ECP-24`

---

## Project Overview

Engineering teams often spend considerable effort performing repetitive defect-triage activities:

- Reading and classifying Jira tickets
- Searching previously resolved incidents
- Mapping defects to repository modules
- Identifying likely root causes
- Recommending remediation approaches
- Locating probable source-code investigation areas
- Preparing defect-resolution and risk reports

This project demonstrates how specialized AI agents can collaborate to automate and standardize that investigation process.

The user provides only a Jira ticket ID, such as:

```text
ECP-24
```

The workflow retrieves the ticket details, invokes the relevant specialist agents, and returns a complete engineering report.

---

## Business Scenario

The project uses a simulated Jira project containing realistic e-commerce defects across the following areas:

- Payment Management
- Shopping Cart Management
- Product Catalog Management
- User Authentication and Security
- Order Processing
- Inventory Management
- Promotion and Coupon Processing
- Product Search

A subset of historical tickets contains documented root causes and resolutions. These records are used as retrieval knowledge for similar-incident analysis.

The Medusa open-source commerce repository is used as the reference application for module and source-code context.

---

## Solution Architecture

```text
User provides Jira Ticket ID
              |
              v
Jira Ticket Retrieval Agent
              |
              v
Bug Classification Agent
              |
              v
Historical Ticket Analysis Agent
              |
              v
Repository Analyzer Agent
              |
              v
Root Cause Analysis Agent
              |
              v
Code Fix Recommendation Agent
              |
              v
Source Code Recommendation Agent
              |
              v
Engineering Report Generator
              |
              v
Final Engineering Defect Resolution Report
```

Detailed architecture documentation is available in:

```text
architecture/workflow-architecture.md
```

---

## Multi-Agent Design

### Agent 0: Jira Ticket Retrieval Agent

Retrieves ticket metadata from the Jira knowledge source.

**Input**

```text
ECP-24
```

**Output**

```json
{
  "ticketId": "ECP-24",
  "title": "JWT token validation failing",
  "description": "Authenticated API requests return Unauthorized responses.",
  "priority": "Highest",
  "epic": "User Authentication & Security",
  "status": "Open"
}
```

---

### Agent 1: Bug Classification Agent

Classifies the defect and identifies its primary business module.

**Responsibilities**

- Determine defect category
- Determine severity
- Identify affected module
- Summarize business impact
- Extract technical keywords

**Example output**

```json
{
  "ticketId": "ECP-24",
  "category": "Authentication Defect",
  "severity": "Critical",
  "affectedModule": "Authentication",
  "businessImpact": "Users cannot access authenticated services.",
  "keywords": [
    "JWT",
    "Unauthorized",
    "Token validation"
  ]
}
```

---

### Agent 2: Historical Ticket Analysis Agent

Searches resolved historical defects for incidents similar to the current ticket.

**Responsibilities**

- Find similar resolved tickets
- Retrieve historical root causes
- Retrieve historical resolutions
- Explain ticket similarity
- Provide a relevance assessment

---

### Agent 3: Repository Analyzer Agent

Maps the classified defect to the relevant Medusa repository module.

**Responsibilities**

- Identify repository folder
- Summarize module responsibilities
- Identify related components
- List common module failure areas
- Provide repository-level investigation context

---

### Agent 4: Root Cause Analysis Agent

Combines ticket details, classification, historical incidents, and repository context to rank probable causes.

**Responsibilities**

- Determine the primary probable root cause
- Rank alternative causes
- Explain the supporting evidence
- Provide a confidence assessment
- Preserve uncertainty where runtime evidence is unavailable

---

### Agent 5: Code Fix Recommendation Agent

Generates proposed engineering changes for the identified root cause.

**Responsibilities**

- Recommend configuration changes
- Recommend application-code changes
- Recommend validation and error-handling improvements
- Recommend security and observability improvements
- Provide deployment and rollback considerations
- Clearly label all recommendations as proposed

The agent does not claim that the fixes were implemented or tested.

---

### Agent 5A: Source Code Recommendation Agent

Uses consolidated Medusa source-code context to identify probable investigation targets.

**Responsibilities**

- Identify relevant repository paths
- Identify candidate services
- Identify candidate classes
- Identify candidate models and repositories
- Identify provider implementations
- Recommend an investigation order
- Distinguish confirmed repository evidence from likely investigation targets

This agent provides source-code investigation guidance. It does not claim to generate a confirmed line-level patch.

---

### Agent 6: Engineering Report Generator

Combines the structured outputs of all specialist agents into the final user-facing report.

**Report sections**

1. Executive Summary
2. Ticket Details
3. Defect Classification
4. Historical Analysis
5. Repository Analysis
6. Source Code Investigation Targets
7. Root Cause Analysis
8. Recommended Fixes
9. Deployment Considerations
10. Rollback Considerations
11. Risk Assessment
12. Next Actions

---

## Workflow Orchestration

The parent orchestrator coordinates all specialist agents.

```text
Step 1: Retrieve the Jira ticket
Step 2: Classify the defect
Step 3: Search historical incidents
Step 4: Map the defect to the repository
Step 5: Perform root cause analysis
Step 6: Generate proposed engineering fixes
Step 7: Identify source-code investigation targets
Step 8: Generate the final engineering report
```

The orchestrator returns only the final report. Intermediate child-agent outputs are not displayed to the end user.

---

## Retrieval-Augmented Generation

The project uses repository and defect knowledge to ground agent responses.

### Jira ticket knowledge

Contains simulated e-commerce epics and bugs with:

- Ticket IDs
- Titles
- Descriptions
- Priorities
- Epic relationships
- Statuses
- Historical root causes
- Historical resolutions

### Repository knowledge

Contains structured context derived from selected Medusa modules:

- Authentication
- Cart
- Payment
- Order
- Inventory
- Product
- Promotion
- Search

### Source-code context

Contains repository-supported information such as:

- Module paths
- Services
- Classes
- Interfaces
- Models
- Providers
- Workflows
- Configuration areas
- Integration areas
- Investigation search terms

---

## Example Use Case

### User input

```text
Analyze ECP-24
```

### Ticket

```text
Title: JWT token validation failing
Priority: Highest
Epic: User Authentication & Security
```

### Workflow result

The workflow generates an Engineering Defect Resolution Report containing:

- Authentication defect classification
- Similar resolved incidents
- Relevant Medusa repository modules
- Probable root cause and alternatives
- Confidence assessment
- Proposed remediation actions
- Candidate source-code investigation paths
- Security considerations
- Deployment guidance
- Rollback considerations
- Risks, assumptions, and next actions

See the sample output:

```text
sample-outputs/ECP24-Final-Report.md
```

---

## Technologies and Concepts

- Microsoft Copilot Studio
- Multi-Agent Orchestration
- Agentic AI
- Retrieval-Augmented Generation
- Prompt Engineering
- Structured JSON handoffs
- Jira defect-management concepts
- Medusa open-source commerce repository
- Root Cause Analysis
- Repository Context Analysis
- Engineering Decision Support
- Human review and approval

---

## Repository Structure

```text
AI-Defect-Resolution-System/
|
|-- README.md
|
|-- architecture/
|   |-- workflow-architecture.md
|   |-- workflow-architecture.png
|
|-- agents/
|   |-- Agent0-Jira-Ticket-Retrieval.md
|   |-- Agent1-Bug-Classification.md
|   |-- Agent2-Historical-Ticket-Analysis.md
|   |-- Agent3-Repository-Analyzer.md
|   |-- Agent4-Root-Cause-Analysis.md
|   |-- Agent5-Code-Fix-Recommendation.md
|   |-- Agent5A-Source-Code-Recommendation.md
|   |-- Agent6-Engineering-Report-Generator.md
|
|-- knowledge-base/
|   |-- All_Jira_Tickets.docx
|   |-- Historical_Resolved_Bugs.docx
|   |-- Medusa_Source_Code_Context_Document.docx
|
|-- sample-inputs/
|   |-- ECP24-Input.json
|
|-- sample-outputs/
|   |-- ECP24-Final-Report.md
|   |-- ECP24-Final-Report.pdf
|
|-- screenshots/
|   |-- orchestrator-agent.png
|   |-- connected-agents.png
|   |-- knowledge-sources.png
|   |-- workflow-input.png
|   |-- workflow-output.png
|
|-- docs/
|   |-- project-overview.md
|   |-- implementation-notes.md
|   |-- future-enhancements.md
|
|-- .gitignore
|-- LICENSE
```

---

## Screenshots

### Orchestrator and connected agents

screenshots/orchestrator-agent.png

screenshots/connected-agents.png

### Knowledge sources

screenshots/knowledge-sources.png

### Workflow result

screenshots/workflow-output.png

> Screenshots will appear after the corresponding image files are added to the `screenshots` folder using the exact file names shown above.

---

## Key Design Decisions

### Single responsibility per agent

Each agent performs one focused task. This reduces responsibility overlap and makes individual agents easier to test and improve.

### Structured handoffs

Specialist agents return structured outputs that can be consumed by downstream agents.

### Evidence-aware recommendations

The workflow distinguishes between:

- Confirmed ticket information
- Historical precedent
- Repository evidence
- Probable root causes
- Proposed recommendations
- Assumptions requiring validation

### Controlled uncertainty

The workflow uses confidence assessments and alternative causes rather than presenting every probable cause as confirmed.

### Human review

The generated report is decision support. Engineering teams should validate runtime evidence, source code, configuration, security impact, and deployment risk before implementing changes.

---

## Current Limitations

- Jira ticket retrieval currently uses a controlled knowledge document rather than a live Jira API integration.
- Repository understanding is derived from prepared module and source-context documentation.
- The workflow does not automatically modify source code.
- The workflow does not automatically create pull requests.
- Code-fix recommendations remain proposed until engineers verify the actual implementation.
- Exact line-level corrections require the relevant source files and runtime evidence.
- Generated outputs may require human review for accuracy and completeness.

---

## Future Enhancements

- Connect directly to Jira Cloud
- Accept a Jira ticket link as workflow input
- Retrieve live ticket comments and attachments
- Integrate directly with GitHub or Azure DevOps repositories
- Retrieve relevant source files dynamically
- Analyze logs and stack traces
- Generate code patches with source-level grounding
- Create draft pull requests with human approval
- Generate automated regression tests
- Add Teams notifications and approval workflows
- Export reports automatically to Word or PDF
- Track agent evaluation metrics
- Add access controls and audit logging

---

## How to Review This Project

Recruiters and reviewers can evaluate the project in this order:

1. Read this README.
2. Review `architecture/workflow-architecture.md`.
3. Open `sample-outputs/ECP24-Final-Report.pdf`.
4. Review screenshots in the `screenshots` folder.
5. Review individual agent documentation in the `agents` folder.
6. Review sanitized knowledge documents in the `knowledge-base` folder.

Microsoft Copilot Studio access is not required to understand the architecture and sample results.

---

## Responsible Use and Data Safety

This repository is intended for learning and portfolio demonstration.

Before publishing:

- Use only synthetic Jira tickets.
- Remove company names, employee names, email addresses, internal URLs, tokens, and credentials.
- Do not upload confidential workplace data.
- Do not upload production logs.
- Do not upload secrets or API keys.
- Do not upload proprietary source code.
- Verify that screenshots do not expose tenant names, account details, internal environment names, or sensitive browser information.
- Respect the license and attribution requirements of referenced open-source projects.

---

## Project Outcome

This project demonstrates practical experience in:

- Designing multi-agent workflows
- Building specialized Copilot Studio agents
- Implementing retrieval-grounded AI behavior
- Passing structured context between agents
- Applying AI to engineering defect investigation
- Producing evidence-aware remediation guidance
- Communicating technical findings through structured reports

---

## Author

**Madhavi Hathwar**

Entry-level Engineering Associate focused on Agentic AI, automation, software quality, repository analysis, and Microsoft Copilot solutions.

---

## Disclaimer

This project is a portfolio proof of concept.

The generated analysis and recommendations are not a substitute for:

- Source-code review
- Runtime diagnostics
- Security review
- QA verification
- Change-management approval
- Production deployment controls

All proposed fixes should be validated by qualified engineers before implementation.