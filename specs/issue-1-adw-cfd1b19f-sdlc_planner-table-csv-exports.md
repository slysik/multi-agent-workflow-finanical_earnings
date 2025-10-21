# Feature: One Click Table Exports

## Metadata
issue_number: `1`
adw_id: `cfd1b19f`
issue_json: `{"number":1,"title":"One click table exports","body":"Using adw_plan_build_review add one click table exports and one click result export feature to get results as csv files.\n\nCreate two new endpoints to support these features. One for exporting tables, one for exporting csv's.   \n\nPlace a download button directly to the left of the 'x' icon for available tables. \n\nPlace a download button directly to the left of the 'hide' button for query results. \n\nUse the appropriate download icon. "}`

## Feature Description
This feature enables users to export both database tables and query results as CSV files with a single click. Users will be able to download complete table data or query results directly from the UI without needing to manually copy data or use external tools. Download buttons will be strategically placed next to existing UI controls for intuitive access - alongside the remove button for tables and next to the hide button for query results. The feature will use appropriate download icons and provide immediate CSV file downloads with proper formatting and headers.

## User Story
As a data analyst or business user
I want to export table data and query results as CSV files with one click
So that I can analyze data in external tools like Excel, share results with colleagues, or archive query outputs for later reference

## Problem Statement
Currently, users can view and query data in the Natural Language SQL Interface but have no way to export this data for use outside the application. Users need to manually copy and paste data from the browser, which is tedious for large datasets, prone to formatting errors, and doesn't preserve column headers or data types properly. This limitation prevents users from leveraging external analysis tools, sharing results with team members who don't have access to the application, or maintaining records of important query results.

## Solution Statement
Implement a comprehensive CSV export system with two new FastAPI endpoints - one for exporting complete table data and another for exporting query results. Add download buttons with appropriate icons to the UI that trigger these endpoints. The solution will generate properly formatted CSV files with headers, handle special characters and data types correctly, and initiate browser downloads with meaningful filenames. The implementation will follow existing security patterns to prevent SQL injection and validate all inputs.

## Relevant Files
Use these files to implement the feature:

**Backend Files:**
- `app/server/server.py` - Add two new API endpoints for table and query result exports
- `app/server/core/data_models.py` - Add Pydantic models for export requests/responses
- `app/server/core/sql_processor.py` - Reuse existing query execution functions
- `app/server/core/sql_security.py` - Use for validating table names and queries
- `app/server/core/file_processor.py` - Reference for understanding current CSV handling patterns

**Frontend Files:**
- `app/client/src/main.ts` - Add download buttons and click handlers for both tables and query results
- `app/client/src/api/client.ts` - Add API client methods for calling export endpoints
- `app/client/src/types.d.ts` - Add TypeScript interfaces for export functionality
- `app/client/src/style.css` - Add styles for download buttons matching existing button patterns
- `app/client/index.html` - No changes needed, buttons will be dynamically created

**Test Reference Files:**
- `.claude/commands/test_e2e.md` - Understanding E2E test structure
- `.claude/commands/e2e/test_basic_query.md` - Example E2E test format

### New Files
- `app/server/core/csv_exporter.py` - New module for CSV export logic and formatting
- `.claude/commands/e2e/test_csv_export.md` - E2E test file for validating export functionality

## Implementation Plan
### Phase 1: Foundation
Create the backend infrastructure for CSV exports including data models, security validation, and the core CSV generation logic. This phase establishes the server-side foundation that both endpoints will utilize.

### Phase 2: Core Implementation
Implement the two API endpoints for table and query result exports, then add the frontend UI components including download buttons with proper icons and event handlers. This phase delivers the main user-facing functionality.

### Phase 3: Integration
Integrate the new export features with existing UI components, ensure proper error handling, add comprehensive logging, and validate the entire flow works correctly with various data types and edge cases.

## Step by Step Tasks
IMPORTANT: Execute every step in order, top to bottom.

### 1. Create CSV Exporter Module
- Create `app/server/core/csv_exporter.py` with functions for converting SQLite query results to CSV format
- Implement `export_table_to_csv(table_name: str)` function that safely queries table data and returns CSV
- Implement `export_query_results_to_csv(query: str, params: list)` function for query result exports
- Add proper CSV formatting with headers, handle special characters, nulls, and various data types
- Include error handling for large datasets and memory management

### 2. Update Data Models
- Open `app/server/core/data_models.py`
- Add `TableExportRequest` Pydantic model with table_name field
- Add `QueryExportRequest` Pydantic model with query and params fields
- Add `CSVExportResponse` model for consistent response structure
- Ensure all models have proper validation and field descriptions

### 3. Implement Table Export Endpoint
- Open `app/server/server.py`
- Add `GET /api/export/table/{table_name}` endpoint
- Validate table_name using `sql_security.validate_identifier()`
- Call `csv_exporter.export_table_to_csv()` to generate CSV data
- Return StreamingResponse with appropriate CSV headers and filename
- Add comprehensive error handling and logging

### 4. Implement Query Export Endpoint
- Continue in `app/server/server.py`
- Add `POST /api/export/query` endpoint accepting query and params
- Validate query using existing security functions
- Call `csv_exporter.export_query_results_to_csv()` with validated inputs
- Return StreamingResponse with CSV data and descriptive filename
- Include error handling for invalid queries and large result sets

### 5. Create E2E Test File
- Create `.claude/commands/e2e/test_csv_export.md` following the pattern from test_basic_query.md
- Include test steps for uploading sample data, exporting a table, running a query, and exporting results
- Add verification steps to check download buttons appear in correct positions
- Specify screenshots for initial state, button placement, and post-download state
- Include success criteria for both export types working correctly

### 6. Add TypeScript Interfaces
- Open `app/client/src/types.d.ts`
- Add `TableExportRequest` and `QueryExportRequest` interfaces matching backend models
- Add type definitions for export response handling
- Ensure interfaces align with Pydantic models for type safety

### 7. Implement API Client Methods
- Open `app/client/src/api/client.ts`
- Add `exportTable(tableName: string)` method that calls GET endpoint
- Add `exportQueryResults(query: string, params: any[])` method for POST endpoint
- Implement blob download handling to trigger browser file downloads
- Add proper error handling and user feedback for failed exports

### 8. Add Download Button Styles
- Open `app/client/src/style.css`
- Add `.download-button` class with consistent styling matching existing buttons
- Include hover effects similar to remove-table-button
- Add appropriate icon styling (using Unicode download arrow or similar)
- Ensure buttons align properly in flexbox layouts

### 9. Add Table Download Buttons
- Open `app/client/src/main.ts`
- Locate table rendering code (around line 288)
- Add download button creation immediately before the remove button
- Use download icon (⬇ or similar Unicode character)
- Add click handler calling `apiClient.exportTable(table.name)`
- Position button to the left of the X button in the table header

### 10. Add Query Results Download Button
- Continue in `app/client/src/main.ts`
- Locate results section rendering (around line 183)
- Add download button creation next to the Hide button
- Store current query and params in closure for export
- Add click handler calling `apiClient.exportQueryResults()`
- Position button to the left of Hide button in results header

### 11. Test CSV Export Functionality
- Start the application using `./scripts/start.sh`
- Upload sample CSV data to create a test table
- Verify download button appears next to X button for the table
- Click table download button and verify CSV file downloads correctly
- Run a natural language query to generate results
- Verify download button appears next to Hide button
- Click results download button and verify CSV exports properly
- Test with various data types, special characters, and large datasets

### 12. Run Validation Commands
Execute every command to validate the feature works correctly with zero regressions.

## Testing Strategy
### Unit Tests
- Test CSV generation with various data types (strings, numbers, dates, nulls)
- Validate proper escaping of special characters (commas, quotes, newlines)
- Test memory efficiency with large datasets
- Verify proper header generation from column names
- Test error handling for invalid table names and queries

### Edge Cases
- Empty tables or query results (should return CSV with headers only)
- Tables with special characters in names
- Columns containing commas, quotes, and newlines
- Very large result sets (>10,000 rows)
- Queries with syntax errors or referencing non-existent tables
- Unicode characters in data
- Null values and empty strings
- Concurrent export requests

## Acceptance Criteria
- Download buttons appear in the specified locations (left of X for tables, left of Hide for results)
- Clicking table download button exports complete table data as CSV
- Clicking query results download button exports current query results as CSV
- CSV files have proper headers matching column names
- Special characters are properly escaped in CSV format
- Downloaded files have descriptive names (e.g., "users_table_export.csv", "query_results_2024_01_15.csv")
- Large exports complete without memory errors or timeouts
- Security validation prevents SQL injection attempts
- Error messages are user-friendly and informative
- All existing functionality continues to work without regression

## Validation Commands
Execute every command to validate the feature works correctly with zero regressions.

- Read `.claude/commands/test_e2e.md`, then read and execute `.claude/commands/e2e/test_csv_export.md` to validate export functionality works
- `cd app/server && uv run pytest` - Run server tests to validate the feature works with zero regressions
- `cd app/server && uv run pytest tests/test_sql_injection.py -v` - Verify security is maintained
- `cd app/client && bun tsc --noEmit` - Run frontend tests to validate the feature works with zero regressions
- `cd app/client && bun run build` - Run frontend build to validate the feature works with zero regressions
- Manual testing: Upload a CSV file, export it, and verify the exported file matches the original data
- Manual testing: Run a complex query with filters and export results to verify filtered data exports correctly

## Notes
- Consider adding a loading spinner for large exports in future iterations
- Future enhancement could include Excel format export option
- May want to add export format options (CSV, TSV, JSON) in a future version
- Download filenames include timestamps to prevent browser caching issues
- StreamingResponse is used to handle large datasets efficiently
- CSV export follows RFC 4180 standard for maximum compatibility