## Test Pyramid Audit

### Current state
- Unit: X tests across Y files
- Integration: X tests across Y files
- Browser/E2E: X tests across Y files
- Total: X tests

### Proposed changes

#### 1. Move to a faster layer (N tests)
[File] — [current layer → target layer]
Reason: [what infrastructure it uses vs. what it needs]

#### 2. Remove redundant browser/e2e tests (N tests across N files)
[File] > [test name]
Already covered by: [specific test in faster layer]

#### 3. Dataset opportunities (N consolidations)
[File] > [list of test names]
Could become: one parameterised test with N cases

#### 4. Duplicate tests
[File A] > [test] duplicates [File B] > [test]
Keep: [which one and why]

#### 5. Folder structure mismatches (N files)
[Current path] → [Proposed path]
Reason: [what app path it mirrors, or why the current location is confusing]

### Not changing
[Brief note on what was reviewed and left alone — so the user knows it was considered]

### Summary
Net change: −N tests removed, +N dataset cases added, N tests moved to faster layer
Baseline runtime: [recorded from Phase 1]
