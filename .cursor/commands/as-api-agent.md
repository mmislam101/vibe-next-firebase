Implement API Agent

**Important:** Approach each change with BDD principles in mind for api layer. Build out the tests first and then ask developer to review before moving onto implementation.

- Code for functions are in the `/functions` folder.
- Refer to `/docs/software/api-specification.md` as context for api changes. Please keep updated as changes occur
- Refer to `/docs/software/testing-behaviors.md` for testing standards and required test behaviors
- Trace the SRD tags that a file implements at the top comments

### Testing Requirements (MANDATORY)
**Every API endpoint MUST have tests before implementation.**

- **Test Location:** Co-located with source in `__tests__/` directories
  - Example: `functions/src/api/v1/user.ts` → `functions/src/api/v1/__tests__/user.test.ts`
- **Test Framework:** Jest with Firebase Functions Test SDK
- **BDD Workflow:**
  1. Write tests first (reference testing-behaviors.md for required behaviors)
  2. Run tests - verify they FAIL (RED phase)
  3. Implement the endpoint
  4. Run tests - verify they PASS (GREEN phase)
  5. Refactor if needed (tests still pass)
- **Required Test Coverage:** See testing-behaviors.md for complete list
  - Authentication (valid token, missing token, invalid token, non-Google provider)
  - Authorization (own data access, other user data blocked)
  - Request validation (valid data, missing fields, invalid types, constraint violations)
  - Success responses (correct status codes and data format)
  - Error handling (404, 500, etc.)
  - Database operations (create, read, update, delete verification)
  - Edge cases (empty strings, null values, special characters, boundaries)
- **Test Execution:** `cd functions && npm test`
- **Failure Handling:**
  - First attempt: Analyze and fix tests or implementation
  - Second attempt: Re-run tests after fixes
  - Still failing: Document issue and request developer review
  - NEVER loop more than twice on failures
- **Documentation:** Update api-specification.md with test status and behaviors covered

### Firestore operations
- Refer and update `/docs/software/database-schema.md` as changes to firestore occur
- Update security rules when adding new collections
- Add indexes for new query patterns
- Use batch writes for related updates
