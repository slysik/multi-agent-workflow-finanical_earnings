# Patch: Change query results export button text from icon to "Export"

## Metadata
adw_id: `d2d0ccbd`
review_change_request: `For 'query Results; make sure download button is to the left of our hide button. Button name needs to be Export.`

## Issue Summary
**Original Spec:** /Users/slysik/tac/tac-6/specs/issue-1-adw-cfd1b19f-sdlc_planner-table-csv-exports.md
**Issue:** Query results export button displays a download arrow icon '⬇' instead of the text "Export"
**Solution:** Change the button innerHTML from '⬇' to 'Export' while maintaining its position to the left of the Hide button

## Files to Modify
Use these files to implement the patch:

- `app/client/src/main.ts` - Update the query results export button text

## Implementation Steps
IMPORTANT: Execute every step in order, top to bottom.

### Step 1: Update query results export button text
- Locate line 225 in `app/client/src/main.ts`
- Change `downloadButton.innerHTML = '⬇';` to `downloadButton.textContent = 'Export';`
- Update the title attribute from 'Export results as CSV' to 'Export query results as CSV'

## Validation
Execute every command to validate the patch is complete with zero regressions.

- `cd app/client && bun tsc --noEmit` - Verify TypeScript compilation succeeds
- `cd app/client && bun run build` - Verify frontend build completes successfully
- Manual test: Run a query and verify the "Export" button appears to the left of the "Hide" button
- Manual test: Click the Export button and verify CSV download works correctly

## Patch Scope
**Lines of code to change:** 2
**Risk level:** low
**Testing required:** Visual verification of button text and position, functional test of export feature