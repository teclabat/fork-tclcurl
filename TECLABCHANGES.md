# TECLAB Changes: main → wip

This document describes all changes made in the `wip` branch compared to `main`.

## Commit History

```
7f09637 new TEA nice/consistent package name
```

## Summary Statistics

```
 69 files changed, 128 insertions(+), 9523 deletions(-)
```

## Major Changes

### 1. Package Name Standardization

The package name has been changed from `TclCurl` to `curl` for better consistency with TEA conventions.

**Impact:**
- Package name: `TclCurl` → `curl`
- Initialization function: `Tclcurl_Init()` → `Curl_Init()`
- All test files updated to use the new package name

**Files Modified:**
- `configure.in`: Changed AC_INIT from "TclCurl" to "curl"
- `generic/tclcurl.c`: Renamed initialization function and updated Tcl_PkgProvide
- All 67 test files: Changed `package require TclCurl` to `package require curl`

### 2. Dynamic Namespace Creation

Added dynamic namespace creation using the package name, replacing hardcoded namespace strings.

**Changes:**
- Introduced `NS_PREFIX` macro: `#define NS_PREFIX PACKAGE_NAME"::"`
- All command registrations now use `NS_PREFIX` instead of hardcoded `"::curl::"`
- Namespace is now explicitly created in the initialization function

**Example:**
```c
// Before:
Tcl_CreateObjCommand(interp, "::curl::init", curlInitObjCmd, ...)

// After:
Tcl_CreateObjCommand(interp, NS_PREFIX "init", curlInitObjCmd, ...)
```

**Files Modified:**
- `generic/tclcurl.c`

### 3. TEA Build System Enhancements

#### New Macro: TEA_PROG_INSTALLED_TCLSH

Added a new autoconf macro to locate the installed tclsh executable (as opposed to the build directory version).

**Purpose:**
- Find the installed tclsh for running tests
- Search in standard installation directories
- Platform-aware executable naming (tclsh86 on Unix, tclsh86.exe on Windows)
- Provides fallback to exec_prefix/bin if not found

**Files:**
- `aclocal.m4`: Added 41 lines implementing the new macro
- `configure.in`: Added call to `TEA_PROG_INSTALLED_TCLSH`
- `Makefile.in`: Added `INSTALLED_TCLSH` variable and uses it for test target

**Impact on Testing:**
```makefile
# Before:
test: binaries libraries
	cd $(srcdir)/tests && $(TCLSH) ...

# After:
test: binaries libraries
	cd $(srcdir)/tests && $(INSTALLED_TCLSH) ...
```

#### Windows Build Improvements

Added explicit linking of Windows Socket library.

**Changes:**
- Added `-lws2_32` to linker flags for Windows builds

**File:** `configure.in`

## Build System Changes

### configure File Removed

The generated `configure` script has been removed from version control (9,445 line deletion).

**Rationale:**
- Generated files should not be tracked in git
- Users should regenerate using autoconf
- Reduces repository size and merge conflicts

### Makefile Updates

**File:** `Makefile.in`

Added support for separate installed tclsh:
```makefile
# Installed tclsh for testing
INSTALLED_TCLSH	= @INSTALLED_TCLSH@
```

## Test File Updates

All 67 test files have been updated to use the new package name.

**Pattern of changes:**
```tcl
# Before:
package require TclCurl

# After:
package require curl
```

**Test files updated:**
- `tests/*.tcl`: 62 files
- `tests/*.test`: 2 files
- `tests/multi/*.tcl`: 5 files

## API Changes

### C API Changes

#### Initialization Function

**Renamed:**
```c
// Old:
EXTERN int Tclcurl_Init(Tcl_Interp *interp)

// New:
EXTERN int Curl_Init(Tcl_Interp *interp)
```

**Note:** This is a breaking change for any C code that directly calls the initialization function.

### Tcl API Changes

#### Package Name

**Changed:**
```tcl
# Old:
package require TclCurl 7.22.1

# New:
package require curl 7.22.1
```

**All commands remain the same:**
- `curl::init`
- `curl::version`
- `curl::escape`
- `curl::unescape`
- `curl::versioninfo`
- `curl::shareinit`
- `curl::easystrerror`
- `curl::sharestrerror`
- `curl::multistrerror`

The namespace prefix remains `curl::` for all commands.

## Backward Compatibility

### Breaking Changes

1. **Package name changed:** Code using `package require TclCurl` must be updated to `package require curl`
2. **C initialization function renamed:** Any C code directly calling `Tclcurl_Init()` must use `Curl_Init()` instead

### Non-Breaking Changes

- All Tcl command names remain unchanged (`curl::*`)
- All command functionality remains the same
- API signatures are unchanged

## Migration Guide

### For Tcl Scripts

Update all package require statements:
```tcl
# Change this:
package require TclCurl

# To this:
package require curl
```

### For C Extensions

Update initialization calls:
```c
// Change this:
Tclcurl_Init(interp);

// To this:
Curl_Init(interp);
```

### For Build Systems

After pulling this branch:
1. Run `autoconf` to regenerate the `configure` script
2. Run `./configure` to regenerate Makefiles
3. Rebuild the package

## Statistics by Component

### Core Changes
- **Build system files:** 3 files modified (configure.in, aclocal.m4, Makefile.in)
- **C source files:** 1 file modified (generic/tclcurl.c)
- **Test files:** 67 files modified
- **Generated files:** 1 file removed (configure)

### Lines of Code
- **Total additions:** 128 lines
- **Total deletions:** 9,523 lines (mostly from configure removal)
- **Net change:** -9,395 lines
