# Pull Request Documentation

## PR Title

```
fix: combine prompt batching with IDE reordering for complete PowerShell fix
```

## PR Target Branch

`next` (per CONTRIBUTING.md - bug fix with UX reorganization)

## PR Description

```markdown
## What

Combined prompt batching with IDE selection reordering to fix PowerShell checkbox/list interactivity issues for all installation paths.

## Why

PowerShell loses TTY state between separate `inquirer.prompt()` calls, causing checkboxes and list prompts to ignore arrow keys. The simple IDE reordering alone only fixed fresh installs, not existing installations where the action menu broke TTY state.

## How

Two essential fixes combined:

1. **Prompt Batching**: For existing installs, batch directory confirm + action menu into single prompt call
2. **IDE Reordering**: Move IDE selection after core config (which primes TTY with batched prompts)

All batching logic localized to `promptInstall()` only; no helper API changes.

Installation flows:

- Fresh install: confirm → core config (batched) → IDE ✅
- Existing install: confirm+menu (batched) → core config (batched) → IDE ✅

## Testing

Tested on PowerShell 5.1 and 7 in Windows Terminal:

- Fresh install to new directory ✅
- Fresh install to existing directory ✅
- Update existing installation ✅
- Reinstall existing installation ✅
- All checkboxes and list prompts respond to arrow keys immediately
```

## Commit Summary

**Commit Hash**: 48804a8
**Type**: fix (bug fix per conventional commits)
**Lines Changed**: +130, -179 (net -49 lines)

## Changes Made

1. **Inlined directory selection loop** in `promptInstall()` with batched prompts (lines 26-108)
2. **Added prompt batching** for existing installations: confirm + action menu in single call (lines 42-64)
3. **Moved IDE selection** from early in flow to after core config (line 127)
4. **Marked legacy helpers** (`getConfirmedDirectory`, `confirmDirectory`) as unused by main flow
5. **No helper API changes** - all fixes localized to `promptInstall()` only

## Key Implementation Details

### Localized Batching Strategy

All PowerShell fixes are contained in `promptInstall()` only:

1. **Directory selection loop** inlined with three cases:
   - Existing BMAD install → batch confirm + action menu
   - Existing directory (no BMAD) → single confirm
   - New directory → single create confirm

2. **Single `inquirer.prompt()` call** per path ensures TTY state preservation

3. **No helper changes** - `confirmDirectory()` and `getConfirmedDirectory()` marked as legacy but unchanged

### Installation Flow Changes

**Before (Broken for existing installs):**

```
1. Directory confirm (separate) ← breaks TTY
2. Action menu (separate) ← breaks TTY
3. Core config (batched)
4. IDE selection ❌ arrows don't work
```

**After (Fixed):**

```
1. Directory confirm + Action menu (batched in one call) ← preserves TTY
2. Core config (batched) ← primes TTY
3. IDE selection ✅ works immediately
```

## Files Changed

- `tools/cli/lib/ui.js` (1 file)

## Testing Checklist

- [x] Arrow keys (↑/↓) navigate checkbox/list items without pressing Enter first
- [x] Space key toggles checkbox selections
- [x] Enter key confirms selection
- [x] No "frozen" or unresponsive prompts
- [x] Installation completes successfully for all paths
- [x] Fresh install path works correctly
- [x] Existing install path works correctly

## PR Checklist

- [x] Code changes complete
- [x] Tested on PowerShell 5.1 and 7
- [x] Commit message follows conventional commits format
- [x] PR description under 200 words (main description is 194 words)
- [x] PR targets `next` branch
- [x] Single focused change (PowerShell TTY fix)

## To Submit PR

When ready to submit, run:

```bash
git push --force-with-lease origin bugfix-powershell-batching
gh pr create --base next --title "fix: combine prompt batching with IDE reordering for complete PowerShell fix" --body "$(cat PR_DOCUMENTATION.md | sed -n '/## What/,/^##/p' | head -n -1)"
```

Or manually create the PR via GitHub web interface targeting the `next` branch.
