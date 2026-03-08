# OpenLP 3.1.7 - macOS Remote Bible Search Crash Fix

**This is a fork of OpenLP 3.1.7 with a bug fix for macOS.**

## License

OpenLP is free open source software distributed under the **GNU General Public License v3 (GPLv3)**.

- **Copyright (c) 2008-2024 OpenLP Developers**
- This fork: Copyright (c) 2026 Jonathan

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see <https://www.gnu.org/licenses/>.

## About This Fork

This fork contains a critical bug fix for macOS where OpenLP crashes when using the remote web interface to search for Bible verses with invalid references.

### The Fix

The fix prevents a crash that occurs because `critical_error_message_box()` was being called from a background thread (HTTP server thread), which is not allowed on macOS.

**File changed:** `openlp/plugins/bibles/lib/db.py`

See `FIX_PATCH.md` for the exact changes.

## Contributing Back

This fix is intended to be submitted to the official OpenLP repository. If accepted, the official version will include this fix.

## Building on macOS

See `BUILD_INSTRUCTIONS.md` for full build instructions.

## Links

- **Official OpenLP Website**: https://openlp.org/
- **Official Repository**: https://gitlab.com/openlp/openlp
- **This Fork**: https://github.com/jonatse/openLPfixesjonatse
