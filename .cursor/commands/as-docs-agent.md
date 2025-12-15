Implement as Doc Agent

# Docs
Documents are broken into three parts,
- System Requirements Specifications (PRD): `docs/system/product-requirements.md`
  - Defines WHAT the system does
  - Requirements are tagged with the `PRD-` prefix
- Software Requirements Document (SRD): `docs/system/software-requirements.md`
  - Defines WHAT the software does
  - Requirements are tagged with the `SRD-` prefix
  - SRD requirements implement one or more PRD requirement.
    - Trace which PRD requirement(s) the SRD requirement implements:
      **SRD-REQ-DB-001:** [Implements: PRD-REQ-PATIENT-001, PRD-REQ-OBS-001]
- Software Development Documents (SDD):
  - Defines HOW the software should behave
  - Software documents implement one or more SRD requirement.
    - Trace which SRD requirement(s) the sections of the software documents:
      [Implements: SRD-REQ-FRONT-006, SRD-REQ-FRONT-007, PRD-SRD-REQ-SEC-001, PRD-REQ-SEC-005]
  - API Doc: `docs/software/api-specification.md`
    - Update whenever endpoints are added/modified/removed
  - Database Doc: `docs/software/database-schema.md`
    - Update whenever collections, fields, or security rules change
  - Frontend Doc: `docs/software/frontend-requirements.md`
    - Update whenever UI components or layouts are added/modified
  - Testing Behavior Doc: `docs/software/testing-behaviors.md`
    - Defines testing behaviors and standards for API endpoints
    - Reference when writing tests for any API implementation

- Avoid adding obvious code for basic examples. 
  - Add code if there's some special implementation details that can only be shown through code
- Don't add implementation status or phase details
- Don't do version or dates
- Keep traceability links updated [Implements: SRD-REQ-XXX]

If Requirements Conflict
1. Document the conflict in your response
2. Propose a resolution based on:
   - Existing architecture patterns
   - Security and performance best practices
   - User experience principles
3. Request developer decision before proceeding

If Requirements Are Unclear
1. State what is unclear
2. Provide 2-3 options based on best practices
3. Request clarification before implementation

Never Assume
- Don't guess at data structures or API contracts
- Don't create placeholder/mock data for non-test code
- Don't skip security or validation requirements
