# Agent Roles
- [Doc Agent](.cursor/commands/as-docs-agent.md)
- [API Agent](.cursor/commands/as-api-agent.md)
- [Frontend Agent](.cursor/commands/as-frontend-agent.md)

**Important:** If no agent role was specified, ask the user to specify, don't assume or default.

# Watefall Method
Agents should follow these steps when making changes. Agent Roles are described in the Agent Roles section.

1. Doc Agent updates the System Requirements Specifications (SRS) based on developer requests
2. Doc Agent makes changes to Software Requirements Document (SRD), based on changes to the SRS.
3. Doc Agent makes changes to the Software Development Documents (SDD) where appropriate
4. API Agent implements the changes to the Database Doc
5. API Agent implements the changes to the API Doc following BDD Principles
  a. API Agent writes test changes first
  b. API Agent runs tests to verify they fail appropriately (red phase)
  c. API Agent implements changes to make tests pass (green phase)
  d. API Agent runs tests again to verify implementation
  e. If tests fail:
    - First attempt: Analyze failure, update tests if they're incorrectly written, or fix implementation
    - Second attempt: Re-run tests after fixes
    - If still failing after 2 attempts: Document the issue and request developer review
    - NEVER loop more than twice on test failures - escalate to developer instead
  f. If tests pass: Proceed to refactoring if needed (refactor phase)
6. Frontend Agent implements the changes to the UI following the Frontend Doc.

## How to Execute the Waterfall

### For Each Developer Request
1. **Identify Scope**: Determine which agents need to be involved
   - UI-only changes? SRS → SRD → Frontend Agent
   - API changes? SRS → SRD → SDD → API Agent (with BDD)
   - Full feature? All agents in sequence

2. **Execute in Order**: Complete each agent's role fully before moving to next
   - Don't skip ahead
   - Don't parallelize (requirements must flow downstream)
