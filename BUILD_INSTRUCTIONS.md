# OpenLP 3.1.7 - macOS Remote Bible Search Crash Fix

## Overview

This is a fork of OpenLP 3.1.7 with a critical fix for a crash that occurs when searching Bible verses via the remote web interface on macOS.

## The Bug

**Issue**: OpenLP crashes on macOS when using the remote web interface to search for Bible verses with an invalid or unrecognized reference.

**Error Message**:
```
EXC_CRASH (SIGABRT) in Thread 32
```

**Crash Details** (from crash report):
- The crash occurred in the HTTP server thread (background thread)
- `QDialog::exec()` was called from a non-main thread
- macOS AppKit throws an exception when trying to create an NSWindow from a background thread
- This triggers `abort()` which crashes the entire application

### Root Cause

The crash occurred in `openlp/plugins/bibles/lib/db.py` in the `get_verses()` method. When searching for a Bible verse:

1. The remote web interface calls the Bible search API
2. The API runs in a background thread (HTTP server thread)
3. If the search fails (e.g., invalid Bible reference), `critical_error_message_box()` is called
4. On macOS, showing a dialog from a background thread causes AppKit to abort
5. The application crashes

## The Fix

The fix adds a thread check before showing any error dialog:

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

### File Changed
- `openlp/plugins/bibles/lib/db.py` (lines 361-372 approximately)

### Behavior After Fix
| Scenario | Before Fix | After Fix |
|----------|-----------|-----------|
| Invalid Bible search from remote (macOS) | **CRASH** | Error logged, app continues |
| Invalid Bible search from main UI | Dialog shown | Dialog shown (unchanged) |
| Valid Bible search | Works | Works (unchanged) |

## Testing the Fix

1. Start OpenLP with web remote enabled
2. Open the remote web interface
3. Search for an invalid Bible reference (e.g., "InvalidBook123")
4. **Before fix**: Application would crash
5. **After fix**: Error is logged, application continues running

## Building on macOS

### Prerequisites

```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install required dependencies
brew install python@3.11 icu4c
brew link icu4c --force
```

### Setup Python Environment

```bash
# Add to PATH (~/.zshrc or ~/.bash_profile)
export PATH="/opt/homebrew/opt/python@3.11/bin:$PATH"
export PATH="/opt/homebrew/opt/icu4c@78/bin:$PATH"
export PKG_CONFIG_PATH="/opt/homebrew/opt/icu4c@78/lib/pkgconfig:$PKG_CONFIG_PATH"

# Install chardet version 4.x (required by OpenLP)
pip3.11 install "chardet<5"

# Install OpenLP in editable mode
pip3.11 install -e .
```

### Running OpenLP

```bash
python3.11 /path/to/run_openlp.py
```

## Original Bug Report

- **macOS version**: macOS (all versions affected)
- **OpenLP version**: 3.1.7
- **Crash thread**: Thread 32 (HTTP server worker thread)
- **Crash location**: QDialog::exec() → QWidget::create() → NSWindow init
- **Trigger**: Remote web API Bible search with invalid reference

## Related Issues

This fix addresses a threading issue specific to macOS where UI operations must occur on the main thread. The same code path works on Linux/Windows because their Qt implementations allow dialogs from background threads, but macOS AppKit does not.

## License

OpenLP is distributed under the GNU General Public License v3. See LICENSE file for details.

## Contributing

This fix is ready to be submitted to the official OpenLP repository. To submit:

1. Fork the OpenLP repository on GitLab
2. Create a branch for this fix
3. Apply the changes to `openlp/plugins/bibles/lib/db.py`
4. Submit a Merge Request

---

**Fixed by**: Jonathan (via GitHub)
**Date**: March 2026
**Original Issue**: macOS crash when using remote Bible search
