# Lilu

macOS kernel extension (C++/C, IOKit) providing kext, process and library patching plus a plugin API.
Targets x86_64 and i386 (`ACID32`). Runs in kernel context: a bug here is a kernel panic.

## Layout
- `Lilu/Sources` – implementation; `Lilu/Headers` – public plugin API (AppleDoc-commented); `Lilu/PrivateHeaders` – internal only.
- `Lilu/Library` – `plugin_start.cpp` and wrappers linked into plugins.
- `capstone`, `hde`, `lzvn`, `sha256`, `umm_malloc` – vendored third-party code; do not edit.
- Key modules: `kern_patcher` (kernel/kext routing), `kern_user` (process/library patching), `kern_mach` (Mach-O parsing, symbols, WP bit), `kern_start` (entry, boot args, config), `kern_api` (plugin registration).

## Build
Requires [MacKernelSDK](https://github.com/acidanthera/MacKernelSDK) checked out next to this repo (see `.github/workflows/main.yml`):

    xcodebuild -jobs 1 -arch x86_64 -arch ACID32 -configuration Debug
    xcodebuild -jobs 1 -arch x86_64 -arch ACID32 -configuration Release
    xcodebuild analyze -quiet -scheme Lilu -configuration Debug   # CI requires zero analyzer findings

No unit tests; verification is build + clang analyzer + loading the kext.

## Conventions
- Tabs for indentation; log with `DBGLOG`/`SYSLOG`/`SYSLOG_COND(cond, "module", ...)`, never raw `IOLog`.
- Use `Buffer::create/deleter`, `evector`, `lilu_os_memcpy/memmove` instead of libc allocators.
- `evector::push_back` returns `bool`, not an index; `erase()` runs the element deleter.
- Never write kernel memory without `MachInfo::setKernelWriting(true, lock)` / `(false, lock)`.
- Treat on-disk Mach-O/prelink/NVRAM data as untrusted: bounds-check offsets and sizes, mind endianness.
- Keep public headers backward compatible; plugins are built against them.

## Git
Conventional commits in English: `feat:`, `fix:`, `chore:`. Version bumps and changelog sync are separate `chore` commits.
