# Fix for macOS Remote Bible Search Crash

## Summary

Fixes a crash that occurs when searching Bible verses via the remote web interface on macOS. The crash happened because `critical_error_message_box()` was being called from a background thread (HTTP server thread), which is not allowed on macOS.

## Changed File

### `openlp/plugins/bibles/lib/db.py`

**Location**: In the `get_verses()` method, around line 361-372

**Before**:
```python
if book_error and show_error:
    critical_error_message_box(
        translate('BiblesPlugin', 'No Book Found'),
        translate('BiblesPlugin', 'No matching book '
                  'could be found in this Bible. Check that you have spelled the name of the book correctly.'))
```

**After**:
```python
if book_error and show_error:
    # Only show error dialog if we're on the main thread
    # This prevents crashes when Bible search is called from remote API (runs in background thread)
    if QtCore.QThread.currentThread() == QtCore.QCoreApplication.instance().thread():
        critical_error_message_box(
            translate('BiblesPlugin', 'No Book Found'),
            translate('BiblesPlugin', 'No matching book '
                      'could be found in this Bible. Check that you have spelled the name of the book correctly.'))
    else:
        # Log the error instead of showing a dialog (safer for background threads)
        log.error('Bible search error: No matching book could be found in this Bible. '
                  'This error occurred in a background thread, so no dialog was shown.')
```

## Explanation

1. **The Problem**: When using the OpenLP remote web interface to search for Bible verses, if the search fails (e.g., invalid reference), the error dialog was being shown from the HTTP server's background thread. On macOS, AppKit requires all UI operations to occur on the main thread, causing a crash.

2. **The Solution**: Add a thread check before showing the dialog. If we're not on the main thread, log the error instead of showing a dialog.

3. **Impact**: 
   - Prevents crash on macOS when using remote Bible search with invalid references
   - Main UI behavior unchanged (dialogs still show when searching from the main application)
   - Remote API errors are logged instead of crashing

## Testing

To verify the fix:
1. Start OpenLP with web remote enabled
2. Use the remote web interface to search for an invalid Bible reference
3. Before fix: App would crash
4. After fix: Error is logged, app continues running
