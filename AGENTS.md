# Docs
Documents are broken into three parts,
- System Requirements Specifications (SRS): `docs/system/system-requirements.md`
  - These contain product details
- Software Requirements Document (SRD): `docs/system/software-requirements.md`
  - These contain architectural level details
- Software Development Documents (SDD):
  - API Doc: `docs/software/api-specification.md`
  - Database Doc: `docs/software/database-schema.md`
  - Frontend Doc: `docs/software/frontend-requirements.md`

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

# Waterfall workflow
1. SRS Agent updates the SRS based on developer requests
2. SRD Agent makes changes to SRD, based on changes to the SRS.
4. SDD Agent makes changes to the Sofware Documents where appropriate
5. API Agent implements the changes to the Database Doc
6. API Agent implements the changes to the API Doc following BDD Principles
  a. Agent writes test changes first. 
  b. Agent implements changes
7. Frontend Agent implements the changes to the UI following the Frontend Doc.
