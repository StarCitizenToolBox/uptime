# Custom Upptime Configuration

This repository uses a customized version of Upptime with enhanced retry logic.

## Changes Made

### 1. Disabled Automatic Template Updates
The `update-template.yml` workflow has been removed to prevent automatic overwriting of custom workflows.

### 2. Custom Retry Logic
Modified the uptime monitoring workflow to implement the following retry behavior:
- When a website error is detected, the system waits 10 seconds before retrying
- The system performs 3 retry attempts (4 total checks including the initial one)
- This prevents false positives caused by temporary network fluctuations

### Implementation Details
The custom retry logic is implemented through:
- `.github/retry-fix.patch`: Patch file that fixes retry bugs and implements the new logic
- `.github/workflows/uptime.yml`: Modified to build uptime-monitor from source with the patch applied

### Retry Logic Changes
1. Fixed bug where `wait()` was called without `await`, causing retries to happen immediately
2. Changed first retry delay from 1s to 10s
3. Changed second retry delay to 10s (was already 10s)
4. Added third retry attempt with 10s delay

### Maintenance
- Do NOT re-enable the `update-template.yml` workflow as it will overwrite these customizations
- When upgrading Upptime versions, the patch file may need to be updated
- Test the workflow after any Upptime version upgrades

## Original Upptime
Based on [Upptime v1.41.0](https://github.com/upptime/upptime)
