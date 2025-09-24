# Tasks: Google Drive Receipt Parser

**Input**: Design documents from `/specs/001-google-drive-receipt-parser/`
**Prerequisites**: plan.md (required), research.md, data-model.md, contracts/

## Execution Flow (main)
```
1. Load plan.md from feature directory
   → If not found: ERROR "No implementation plan found"
   → Extract: tech stack, libraries, structure
2. Load optional design documents:
   → data-model.md: Extract entities → model tasks
   → contracts/: Each file → contract test task
   → research.md: Extract decisions → setup tasks
3. Generate tasks by category:
   → Setup: project init, dependencies, linting
   → Tests: contract tests, integration tests
   → Core: models, services, CLI commands
   → Integration: DB, middleware, logging
   → Polish: unit tests, performance, docs
4. Apply task rules:
   → Different files = mark [P] for parallel
   → Same file = sequential (no [P])
   → Tests before implementation (TDD)
5. Number tasks sequentially (T001, T002...)
6. Generate dependency graph
7. Create parallel execution examples
8. Validate task completeness:
   → All contracts have tests?
   → All entities have models?
   → All endpoints implemented?
9. Return: SUCCESS (tasks ready for execution)
```

## Format: `[ID] [P?] Description`
- **[P]**: Can run in parallel (different files, no dependencies)
- Include exact file paths in descriptions

## Path Conventions
- **Single project**: `src/`, `tests/` at repository root
- **Web app**: `backend/src/`, `frontend/src/`
- **Mobile**: `api/src/`, `ios/src/` or `android/src/`
- Paths shown below assume single project - adjust based on plan.md structure

## Phase 3.1: Setup
- [ ] T001 Create project structure per implementation plan with Vite
- [ ] T002 Initialize JavaScript project with Vite and minimal dependencies
- [ ] T003 [P] Configure linting and formatting tools (ESLint, Prettier)
- [ ] T004 [P] Set up folder structure (src/components/, src/services/, src/utils/, src/models/)

## Phase 3.2: Tests First (TDD) ⚠️ MUST COMPLETE BEFORE 3.3
**CRITICAL: These tests MUST be written and MUST FAIL before ANY implementation**
- [ ] T005 [P] Contract test for authentication API in tests/contract/test-auth-api.js
- [ ] T006 [P] Contract test for Google Drive API in tests/contract/test-drive-api.js
- [ ] T007 [P] Contract test for receipt management API in tests/contract/test-receipt-api.js
- [ ] T008 [P] Contract test for export API in tests/contract/test-export-api.js
- [ ] T009 [P] Integration test for receipt processing in tests/integration/test-receipt-processing.js
- [ ] T010 [P] Integration test for Google Drive integration in tests/integration/test-drive-integration.js

## Phase 3.3: Core Implementation (ONLY after tests are failing)
- [ ] T011 [P] Receipt model in src/models/receipt.js based on data-model.md
- [ ] T012 [P] Database service with SQLite operations in src/services/database-service.js
- [ ] T013 [P] Google Drive service with OAuth integration in src/services/drive-service.js
- [ ] T014 [P] OCR service with Tesseract.js integration in src/services/ocr-service.js
- [ ] T015 Authentication API endpoints implementation
- [ ] T016 Google Drive API endpoints implementation
- [ ] T017 Receipt management API endpoints implementation
- [ ] T18 Export API endpoints implementation

## Phase 3.4: Integration
- [ ] T019 Connect database service to receipt model
- [ ] T020 Integrate OCR service with drive service for receipt processing
- [ ] T021 Implement receipt processing workflow with status tracking
- [ ] T022 Add authentication middleware to API endpoints

## Phase 3.5: Polish
- [ ] T023 [P] Unit tests for validation helpers in tests/unit/test-validators.js
- [ ] T024 [P] Unit tests for utility functions in tests/unit/test-utils.js
- [ ] T025 [P] Unit tests for receipt model in tests/unit/test-receipt-model.js
- [ ] T026 Performance optimization for large receipt sets
- [ ] T027 [P] Update documentation in docs/api.md
- [ ] T028 Accessibility improvements
- [ ] T029 Error handling and user feedback
- [ ] T030 Final testing and bug fixes

## Dependencies
- Tests (T005-T010) before implementation (T011-T018)
- T011 (Receipt model) blocks T012 (Database service integration)
- T012 (Database service) blocks T019
- T013 (Drive service) blocks T020
- T014 (OCR service) blocks T020
- T015-T018 (API implementations) blocks T022
- Implementation before polish (T023-T030)

## Parallel Example
```
# Launch T005-T010 together:
Task: "Contract test for authentication API in tests/contract/test-auth-api.js"
Task: "Contract test for Google Drive API in tests/contract/test-drive-api.js"
Task: "Contract test for receipt management API in tests/contract/test-receipt-api.js"
Task: "Contract test for export API in tests/contract/test-export-api.js"
Task: "Integration test for receipt processing in tests/integration/test-receipt-processing.js"
Task: "Integration test for Google Drive integration in tests/integration/test-drive-integration.js"
```

## Notes
- [P] tasks = different files, no dependencies
- Verify tests fail before implementing
- Commit after each task
- Avoid: vague tasks, same file conflicts

## Task Generation Rules
*Applied during main() execution*

1. **From Contracts**:
   - Each contract file → contract test task [P]
   - Each endpoint → implementation task
   
2. **From Data Model**:
   - Each entity → model creation task [P]
   - Relationships → service layer tasks
   
3. **From User Stories**:
   - Each story → integration test [P]
   - Quickstart scenarios → validation tasks

4. **Ordering**:
   - Setup → Tests → Models → Services → Endpoints → Polish
   - Dependencies block parallel execution

## Validation Checklist
*GATE: Checked by main() before returning*

- [ ] All contracts have corresponding tests
- [ ] All entities have model tasks
- [ ] All tests come before implementation
- [ ] Parallel tasks truly independent
- [ ] Each task specifies exact file path
- [ ] No task modifies same file as another [P] task