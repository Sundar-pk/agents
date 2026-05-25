```markdown
# Angular-20-Migration Agent (migration-agents.md)

## Overview
- **Purpose**: Automatically migrate Angular framework components from module-based to standalone architecture for Angular 20 upgrade. Converts components one-by-one, updates syntax (ngIf/ngClass/ngStyle to modern equivalents), removes module files, updates public APIs and imports, validates with unit tests after each component, and auto-fixes test failures before proceeding to next component.
- **Primary user**: Technical architect and development team managing the framework migration.
- **Target**: 50+ components in the workspace.

## Scope & limitations
- **Scope**: 
  - Convert components to standalone (standalone: true).
  - Migrate template syntax: ngIf → @if/@else, [ngClass] → [class], [ngStyle] → [style].
  - Remove module files (.module.ts).
  - Update public_api.ts to remove module exports.
  - Find and replace module imports across workspace with standalone component imports.
  - Run `npm run test` after each component migration.
  - Auto-fix test failures and retry until tests pass.
  - Iterate through all components in the list sequentially.
- **Not covered**: 
  - E2E tests (only unit tests validated).
  - Manual code review sign-off (agent auto-proceeds on green tests).
  - Rollback strategy (assumes version control handles rollback).
  - Migration of services or interceptors (focus: components only).

## Inputs
- **Component list**: Array of component paths/names to migrate (e.g., `['button', 'card', 'modal', ...]`).
- **Workspace structure**: Root path, component folder convention, module file naming pattern, public_api.ts location.
- **Test command**: `npm run test` (configurable).
- **Repo context**: Git branch, commit before migration starts.
- **Config**: 
  - Max retry attempts for test fix (default: 3).
  - Auto-commit after each successful component (true/false).
  - Logging level, output directory for migration logs.

## Outputs
- **Migration summary per component**: 
  - Component name.
  - Status: Migrated & tests passed / Failed & skipped / Retried N times.
  - Changes made: files modified, lines changed, syntax transformations count.
  - Test results: pass/fail, error logs (if any).
  - Fix attempts: description of auto-fixes applied.
- **Final migration report**: 
  - Total components processed.
  - Success count, failure count, skipped count.
  - Total time, commits generated.
  - List of failed components requiring manual intervention.
- **Artifacts**: 
  - Migration log file per component.
  - Commit SHAs for each successful migration (if auto-commit enabled).
  - Diff files for review.

## Behavior & decision logic (sequential workflow)

### Main loop (for each component in list)
1. **Select next component** from the list.
2. **Perform migration steps** (details below).
3. **Run `npm run test`**.
4. **If tests pass**: 
   - Mark component as migrated.
   - Optionally commit changes with message `chore: migrate {component} to standalone (Angular 20)`.
   - Proceed to next component.
5. **If tests fail**: 
   - Analyze test errors.
   - Apply auto-fix strategies (details below).
   - Increment retry counter.
   - Re-run `npm run test`.
   - Repeat fix-and-test loop up to max retry attempts.
6. **If max retries exceeded**: 
   - Mark component as failed.
   - Log errors and skip to next component.
   - Flag for manual intervention.
7. **Continue until all components processed**.
