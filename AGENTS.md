# Waterfall workflow (Read this first!)
Agents should follow these steps when making changes. Agent Roles are described in the Agent Roles section.

1. SRS Agent updates the System Requirements Specifications (SRS) based on developer requests
2. SRD Agent makes changes to Software Requirements Document (SRD), based on changes to the SRS.
3. SDD Agent makes changes to the Software Development Documents (SDD) where appropriate
4. API Agent implements the changes to the Database Doc
5. API Agent implements the changes to the API Doc following BDD Principles
  a. Agent writes test changes first
  b. Agent runs tests to verify they fail appropriately (red phase)
  c. Agent implements changes to make tests pass (green phase)
  d. Agent runs tests again to verify implementation
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

## Handling Conflicts and Ambiguity

### If Requirements Conflict
1. Document the conflict in your response
2. Propose a resolution based on:
   - Existing architecture patterns
   - Security and performance best practices
   - User experience principles
3. Request developer decision before proceeding

### If Requirements Are Unclear
1. State what is unclear
2. Provide 2-3 options based on best practices
3. Request clarification before implementation

### Never Assume
- Don't guess at data structures or API contracts
- Don't create placeholder/mock data for non-test code
- Don't skip security or validation requirements

# Docs
Documents are broken into three parts,
- System Requirements Specifications (SRS): `docs/system/system-requirements.md`
  - Update whenever new features or functional requirements are requested  
- Software Requirements Document (SRD): `docs/system/software-requirements.md`
  - Update whenever SRS changes OR architectural decisions are made
- Software Development Documents (SDD):
  - API Doc: `docs/software/api-specification.md`
    - Update whenever endpoints are added/modified/removed
  - Database Doc: `docs/software/database-schema.md`
    - Update whenever collections, fields, or security rules change
  - Frontend Doc: `docs/software/frontend-requirements.md`
    - Update whenever UI components or layouts are added/modified

- Don't add implementation status or phase details
- Don't do version or dates
- Keep traceability links updated [Implements: SRD-REQ-XXX]

# Agent Roles

## SRS Agent
- SRS Agent maintains the System Requirements Specifications (SRS): `docs/system/system-requirements.md`
- Requirements are tagged with the `SRS-` prefix

## SRD Agent
- SRD Agent maintains the Software Requirements Document (SRD): `docs/system/software-requirements.md`
- Requirements are tagged with the `SRD-` prefix
- SRD requirements implement one or more SRS requirement.
  - Trace which SRS requirement(s) the SRD requirement implements:
    **SRD-REQ-DB-001:** [Implements: SRS-REQ-PATIENT-001, SRS-REQ-OBS-001]

## SDD Agent
- Software Agent maintains documents in `docs/software/` directory:
  - API specs: `docs/software/api-specification.md`
  - Front-end specs `docs/software/frontend-requirements.md`
  - Database specs `docs/software/database-schema.md`
- Software documents implement one or more SRD requirement.
  - Trace which SRD requirement(s) the sections of the software documents:
    [Implements: SRD-REQ-FRONT-006, SRD-REQ-FRONT-007, SRS-SRD-REQ-SEC-001, SRS-REQ-SEC-005]

## API Agent
**Important:** Approach each change with BDD principles in mind for api layer. Build out the tests first and then ask developer to review before moving onto implementation.

- Code for functions are in the `/functions` folder.
- Refer to `/docs/software/api-specification.md` as context for api changes. Please keep updated as changes occur
- Trace the SRD tags that a file implements at the top comments

### Firestore operations
- Refer and update `/docs/software/database-schema.md` as changes to firestore occur
- Update security rules when adding new collections
- Add indexes for new query patterns
- Use batch writes for related updates

## Frontend Agent
- Refer to `docs/software/frontend-requirements.md` as context for UI changes. Please keep updated as changes occur
- Don't do tests for UI
- Trace the SRD tags that a file implements at the top comments
