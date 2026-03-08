# OpenLP with macOS Threading Fix

This is a fork of OpenLP 3.1.7 with a fix for a crash that occurs when searching Bible verses via the remote web interface on macOS.

## The Bug

When using the OpenLP remote web interface to search for Bible verses, OpenLP would crash on macOS with an error like:

```
EXC_CRASH (SIGABRT) in Thread 32
```

### Root Cause

The crash occurred because `critical_error_message_box()` was being called from a background thread (the HTTP server thread) when searching for Bible verses via the remote web interface. On macOS, showing a dialog from a background thread causes AppKit to abort, since all UI operations must happen on the main thread.

## The Fix

The fix is in `openlp/plugins/bibles/lib/db.py` in the `get_verses()` method. It now checks if we're on the main thread before showing the error dialog:

```python
if book_error and show_error:
    # Only show error dialog if we're on the main thread
    # This prevents crashes when Bible search is called from remote API (runs in background thread)
    if QtCore.QThread.currentThread() == QtCore.QCoreApplication.instance().thread():
        critical_error_message_box(...)
    else:
        # Log the error instead of showing a dialog (safer for background threads)
        log.error('Bible search error: No matching book could be found...')
```

## Building on macOS

### Prerequisites

Install Homebrew (if not already installed):
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Install required dependencies:
```bash
brew install python@3.11 icu4c
brew link icu4c --force
```

### Setup Python Environment

```bash
# Add to PATH (add to ~/.zshrc or ~/.bash_profile)
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
python3.11 /path/to/openlp/run_openlp.py
```

Or with the web remote enabled:
```bash
python3.11 /path/to/openlp/run_openlp.py -w
```

## Testing the Fix

To verify the fix works:

1. Start OpenLP with web remote: `python3.11 run_openlp.py -w`
2. Open the remote web interface (http://localhost:8008)
3. Try searching for an invalid Bible reference (e.g., "InvalidBook123")
4. **Before fix**: App would crash
5. **After fix**: Error is logged, app continues running

## Original Bug Report

- macOS crash when searching Bible verses via remote web interface
- OpenLP version: 3.1.7
- The crash occurred in Thread 32 when QDialog::exec() was called from a background thread

## License

OpenLP is distributed under the GNU General Public License v3. See LICENSE file for details.
