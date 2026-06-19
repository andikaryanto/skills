# Bug Fix Workflow

This document describes the workflow for fixing bugs.

## Goal

Identify the root cause of the bug and implement the smallest safe fix.

Avoid rewriting unrelated code.

## When to Use

Use this workflow when:

- Fixing incorrect behavior.
- Fixing validation issues.
- Fixing API response issues.
- Fixing business logic defects.
- Fixing database query issues.

Examples:

- Product price calculation is incorrect.
- Pagination returns duplicate records.
- Validation allows invalid data.
- API returns an incorrect response.

## Understand the Bug

Before changing code, identify:

- Expected behavior.
- Actual behavior.
- Steps to reproduce.
- Affected endpoints or services.

If the bug report is incomplete, request additional information.

## Locate the Root Cause

Inspect the relevant layers:

- Controller.
- Middleware.
- Service.
- Query.
- Repository.
- Model.

Do not implement a fix until the root cause is identified.

## Add or Update Tests

Create a test that reproduces the bug.

The test should fail before the fix.

Examples:

- Feature test.
- Service test.
- Query test.

## Implement the Fix

Apply the smallest change necessary to fix the bug.

Avoid:

- Unrelated refactoring.
- Architectural changes without justification.
- Rewriting working code.

## Verify the Fix

Confirm:

- The failing test now passes.
- Existing tests still pass.
- Expected behavior matches requirements.

## Review Checklist

Before completing the fix, ensure:

- Root cause was identified.
- A test reproduces the bug.
- The fix is minimal and targeted.
- Existing architecture rules are respected.
- No unrelated code was modified.
- All relevant tests pass.
