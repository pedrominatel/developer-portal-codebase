---
name: esp-idf-v6-migration
description: Migrate existing ESP-IDF application repositories from ESP-IDF 5.x to 6.0 using cumulative migration guides, version detection, build failure triage, and subsystem-specific fixes.
compatibility: Works with standard ESP-IDF command-line tools and official ESP-IDF migration guides.
---

# ESP-IDF v6 Migration

Use this skill when upgrading an existing ESP-IDF application repository from ESP-IDF 5.x to 6.0.

This skill is optimized for application repos, not standalone reusable components. Treat migration as cumulative: if the project starts before 5.5, review each intermediate migration guide up to 5.5 before applying the 5.5 to 6.0 migration guide.

## When to Use

- This migration skill is not recommended for projects that are based on older than ESP-IDF v4.4, as the migration path is too complex
- Migrating an application from ESP-IDF 5.5 to 6.0
- Migrating an application from 5.4, 5.3, 5.2, 5.1, or 5.0 by chaining intermediate 5.x guides first
- Migrating from 4.4 application forward through 5.0 and then cumulatively to 6.0
- Triaging build failures after switching the project to ESP-IDF 6.0
- Auditing API removals, driver moves, toolchain changes, and Kconfig/build-system breakages during migration

## Source Priority

- Use the official online ESP-IDF migration guides as the primary source of truth.
- Use a local ESP-IDF 6.0 checkout as supporting context for 6.0-only topics, file paths, and examples when one is available.
- Find a local install by running `eim list` first. If EIM is not available or the correct 6.0 install is still unclear, check `IDF_PATH` and the active `idf.py` environment. If the correct 6.0 checkout still cannot be identified reliably, ask the user to provide the path.
- For projects starting on older 5.x versions, do not compress the older migration history into guesses. Open the relevant official intermediate guides listed in [Migration Paths](#migration-paths).

## Workflow

### 1. Detect the current project baseline

- Confirm the current ESP-IDF version before proposing changes.
- Check `idf.py --version`, `eim list`, exported `IDF_PATH`, project docs, CI config, and lockfiles as needed.
- Confirm the project target and whether runtime validation on hardware is expected.
- If the starting version is unclear, stop and resolve that first.

### 2. Pick the cumulative migration path

- Use [Migration Paths](#migration-paths) to select the exact guide chain.
- Required rule:
  - 5.5 starts with the 5.5 to 6.0 guide
  - 5.4 and earlier must review every intermediate migration guide through 5.5, then 6.0
- Use [Version Coverage Summary](#version-coverage-summary) to decide which subsystem chapters to inspect first.

### 3. Switch the environment to the target IDF

- Move the app into an ESP-IDF 6.0 environment before judging compatibility.
- Confirm Python, CMake, and toolchain requirements before the first build.
- During the first build on ESP-IDF 6.0, check the log for CMake version warnings before triaging deeper compile failures.
- If the top-level `CMakeLists.txt` still declares an older minimum, update it to `cmake_minimum_required(VERSION 3.22)` as an early build-system migration step.
- Prefer running the first build without broad source edits so the failure set reflects real migration blockers.

### 4. Run an initial build and categorize failures

- Build once on the target ESP-IDF version.
- Review warnings as well as hard failures during this first build, especially CMake baseline warnings that should be resolved before subsystem-level edits.
- Group failures before editing:
  - build system or linker
  - GCC or header/toolchain
  - removed or moved components
  - components not suppoprted
  - driver dependency splits
  - subsystem API changes
  - config and Kconfig syntax issues
- Use [Common Migration Breakpoints](#common-migration-breakpoints) to map symptoms to likely migration areas.
- Use [Build Error Symptom Map](#build-error-symptom-map) when the build log already points to a concrete missing header, removed API, linker failure, or tool behavior change.

### 5. Fix migration issues by subsystem

- Build system:
  - check CMake version warnings and align the top-level `cmake_minimum_required(...)` with `3.22` when the project still pins an older baseline
  - check orphan section errors
  - check constructor-order assumptions
  - check Kconfig v3 syntax compatibility
  - check warnings-as-errors defaults
- Toolchain:
  - check GCC 15 warnings and promoted errors
  - replace outdated headers such as `sys/dirent.h` where needed
  - account for Picolibc header differences if enabled
- Components and dependencies:
  - replace removed in-tree drivers with Component Registry dependencies where required
  - update `REQUIRES` and `PRIV_REQUIRES` explicitly when legacy transitive dependencies disappear
  - review `driver` usage and split dependencies to `esp_driver_*` components when needed
- Subsystems:
  - review the relevant migration chapters for networking, peripherals, protocols, Wi-Fi, storage, system, security, provisioning, and Bluetooth
  - do not apply mechanical edits outside the subsystem actually implicated by build or runtime evidence
  - do not edit external components or apply any patches. Patches can only be appied if the user explicitly confirms to do so

### 6. Rebuild and validate behavior

- Rebuild after each coherent migration batch, not after every small edit.
- If firmware behavior changed and hardware is available, flash and capture monitor logs on the target board.
- Compare runtime behavior against the pre-migration app expectations, especially for networking, storage, and peripheral initialization.
- Report what was validated and what was not.

## References

- Migration path matrix: [Migration Paths](#migration-paths)
- Common migration breakpoints: [Common Migration Breakpoints](#common-migration-breakpoints)
- Build error symptom map: [Build Error Symptom Map](#build-error-symptom-map)
- Version coverage summary: [Version Coverage Summary](#version-coverage-summary)

## Avoid

- Jumping straight from 5.4 or earlier to 6.0 without reviewing intermediate migration guides
- Treating a local ESP-IDF 6.0 checkout as more authoritative than the current official migration docs
- Suppressing warnings or linker errors before understanding the underlying breakage
- Assuming legacy `driver` dependencies or moved components still arrive transitively
- Making broad unrelated cleanups while the migration error set is still being established

## Migration Paths

Use the official online migration guides as the primary source of truth. For 6.0-specific context, a local ESP-IDF 6.0 checkout can help locate examples and docs, but it does not replace the official guide chain. Run `eim list` first to discover installed ESP-IDF versions. If EIM is unavailable or the right 6.0 tree is still unclear, check `IDF_PATH` and the active `idf.py` environment. If the correct local 6.0 tree still cannot be identified reliably, ask the user to provide the path.

### Rule

If the application starts before 5.5, review every intermediate migration guide through 5.5 before applying the 5.5 to 6.0 guide.

### Starting Versions

- 5.5
  - Review: 5.5 to 6.0
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-6.x/6.0/index.html

- 5.4
  - Review: 5.4 to 5.5, then 5.5 to 6.0
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.5/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-6.x/6.0/index.html

- 5.3
  - Review: 5.3 to 5.4, 5.4 to 5.5, then 5.5 to 6.0
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.4/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.5/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-6.x/6.0/index.html

- 5.2
  - Review: 5.2 to 5.3, 5.3 to 5.4, 5.4 to 5.5, then 5.5 to 6.0
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.3/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.4/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.5/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-6.x/6.0/index.html

- 5.1
  - Review: 5.1 to 5.2, 5.2 to 5.3, 5.3 to 5.4, 5.4 to 5.5, then 5.5 to 6.0
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.2/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.3/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.4/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.5/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-6.x/6.0/index.html

- 5.0
  - Review: 5.0 to 5.1, 5.1 to 5.2, 5.2 to 5.3, 5.3 to 5.4, 5.4 to 5.5, then 5.5 to 6.0
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.1/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.2/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.3/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.4/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.5/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-6.x/6.0/index.html

- 4.4
  - Review: 4.4 to 5.0, 5.0 to 5.1, 5.1 to 5.2, 5.2 to 5.3, 5.3 to 5.4, 5.4 to 5.5, then 5.5 to 6.0
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.0/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.1/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.2/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.3/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.4/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.5/index.html
  - https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-6.x/6.0/index.html

## Common Migration Breakpoints

Use this section to map a failure symptom to the migration area to inspect next. Open the official migration guide chapter for the implicated version and subsystem before making changes.

### Build System

- Linker orphan section errors appear in 6.0 and often point to linker fragment or placement issues.
- Constructor execution order changed in 6.0 due to `__libc_init_array()`, which can break registration logic that assumed reverse ordering.
- Kconfig syntax changes matter in newer versions because ESP-IDF 6.0 uses esp-idf-kconfig v3.
- Default compiler warnings as errors in 6.0 can turn previously tolerated issues into hard build failures.

### GCC and Toolchain

- GCC changed repeatedly across 5.x and 6.0. Treat new warnings as migration work, not random noise.
- GCC 15 in 6.0 can expose include-guard issues, unterminated string initializers, and stricter header usage.
- Header assumptions may break:
  - use `<dirent.h>` instead of relying on `<sys/dirent.h>` for function prototypes
  - use `<signal.h>` instead of `<sys/signal.h>` when Picolibc is enabled
- Check Python and CMake baselines when the environment itself fails before application code builds.

### Components and Dependencies

- Legacy transitive dependencies become less reliable as ESP-IDF moves toward explicit component ownership.
- The legacy `driver` component no longer carries the same public dependencies in 6.0. Add the needed `esp_driver_*` dependencies explicitly.
- Some drivers and tools move out of the core tree and must be pulled in from the ESP Component Registry.
- If a previously in-tree feature disappears, check whether Espressif moved it to a standalone component before replacing it manually.

### Peripheral APIs

- Peripheral migrations often break at compile time because enums, config fields, or driver component boundaries changed.
- Projects that still depend on legacy drivers should be reviewed carefully before removing `driver`; some migrations require temporary coexistence plus explicit new dependencies.
- GPIO-sharing behavior and loopback-related assumptions may need review in newer peripheral drivers.

### Networking, Protocols, and Wi-Fi

- Removed headers or deprecated APIs are common breakpoints:
  - SNTP header changes
  - ping API removals
  - Wi-Fi enum and API removals or renames
  - ESP-NETIF iteration helper changes
- Check registry-backed or moved protocol components when a library appears to have vanished from the core IDF tree.
- If the build passes but runtime behavior changes, verify DHCP, DNS, Wi-Fi startup, and socket-related code paths first.

### System, Storage, and Security

- NVS, partition behavior, security defaults, or system APIs may shift over multiple 5.x releases, not only in 6.0.
- When the app depends on boot order, constructors, startup hooks, or low-level init flow, review system chapters for each intermediate migration guide.
- If storage or security failures appear only after boot, do not assume they are unrelated to the version jump.

### Tools and Environment

- ESP-IDF 6.0 raises environment expectations:
  - Python 3.10 or newer
  - CMake 3.22 or newer
- The first build may warn about an outdated project CMake baseline before compile errors become the main blockers.
- When that warning appears, align the top-level `cmake_minimum_required(...)` in `CMakeLists.txt` with `3.22` before deeper migration triage.
- `idf.py size` output format changes can break scripts that parse legacy JSON.
- `idf.py efuse*` commands now require an explicit serial port.
- If a CI workflow breaks after the version bump, audit the toolchain and helper command usage before changing application code.

## Build Error Symptom Map

Use this section when the first build on a newer ESP-IDF version fails and you need a concrete next step. Treat these mappings as triage guidance, then open the relevant official migration chapter before editing code.

### Linker and Build-System Errors

#### Symptom

`orphan section` linker errors during the final link step.

#### Check Next

- ESP-IDF 6.0 build-system migration guide online
- Custom linker fragments
- Recently added attributes, sections, or linker script changes

#### Likely Fix Direction

- Remove unused code or data that lands in an unplaced section
- Add or fix a linker fragment so the section is placed explicitly
- Avoid downgrading `CONFIG_COMPILER_ORPHAN_SECTIONS` unless you understand why the section is orphaned

### Constructor and Startup Ordering

#### Symptom

Tests, registries, or startup-time tables initialize in the wrong order after the version bump.

#### Check Next

- ESP-IDF 6.0 build-system migration guide
- Code using constructors or registration macros

#### Likely Fix Direction

- Audit assumptions about reverse constructor ordering
- Convert list insertion logic from head insertion to tail insertion where appropriate
- Use constructor priorities when ordering must be explicit

### Kconfig and Configuration Parsing

#### Symptom

`Kconfig` parsing errors, invalid syntax, or config options that no longer behave as expected.

#### Check Next

- ESP-IDF 6.0 build-system migration guide
- esp-idf-kconfig v2 to v3 migration guide

#### Likely Fix Direction

- Update `Kconfig` syntax for esp-idf-kconfig v3 compatibility
- Remove outdated constructs instead of working around parser failures
- Re-run reconfigure after fixing project or component config definitions

### Warnings Promoted to Errors

#### Symptom

Build breaks on warnings that did not fail before.

#### Check Next

- ESP-IDF 6.0 build-system and toolchain migration guides
- Exact GCC warning names in the build log

#### Likely Fix Direction

- Fix the code first where practical
- Use targeted suppression only when the warning is understood and the code is still valid
- Avoid broad suppression as the first response

### `sys/dirent.h` Function Declaration Failures

#### Symptom

Errors such as:

```c
implicit declaration of function 'opendir'
```

when the source includes `<sys/dirent.h>`.

#### Check Next

- ESP-IDF 6.0 toolchain migration guide

#### Likely Fix Direction

- Replace `<sys/dirent.h>` with `<dirent.h>`

### Picolibc `sys/signal.h` Missing

#### Symptom

Errors such as:

```c
fatal error: sys/signal.h: No such file or directory
```

with Picolibc enabled.

#### Check Next

- ESP-IDF 6.0 toolchain migration guide
- Project libc configuration

#### Likely Fix Direction

- Replace `<sys/signal.h>` with `<signal.h>`

### Legacy `driver` Dependency Breakages

#### Symptom

Build errors after a component or app still depends on `driver`, but symbols or headers from specific drivers are no longer available transitively.

#### Check Next

- ESP-IDF 6.0 peripherals migration guide
- Component `CMakeLists.txt`
- Main project `CMakeLists.txt`

#### Likely Fix Direction

- Keep `driver` only where legacy APIs are still required
- Add explicit `esp_driver_*` dependencies for the actual peripherals in use
- Update `REQUIRES` and `PRIV_REQUIRES` instead of assuming old transitive behavior

### Removed or Moved Ethernet Driver APIs

#### Symptom

Missing Ethernet PHY or SPI MAC constructors such as `esp_eth_phy_new_*` or `esp_eth_mac_new_*`.

#### Check Next

- ESP-IDF 6.0 networking migration guide
- Whether the project now needs a Component Registry dependency

#### Likely Fix Direction

- Add the replacement Ethernet driver package from the ESP Component Registry
- Update includes and component dependencies to match the externalized driver layout

### `sntp.h` Missing

#### Symptom

Build fails because `sntp.h` no longer exists.

#### Check Next

- ESP-IDF 6.0 networking migration guide

#### Likely Fix Direction

- Replace `sntp.h` with `esp_sntp.h`

### Ping API Removal

#### Symptom

Missing headers or symbols for:

- `esp_ping.h`
- `ping.h`
- `ping_init`
- `ping_deinit`
- `esp_ping_set_target`
- `esp_ping_get_target`
- `esp_ping_result`

#### Check Next

- ESP-IDF 6.0 networking migration guide

#### Likely Fix Direction

- Migrate to the socket-based ping API from `ping/ping_sock.h`

### Wi-Fi API or Enum Removals

#### Symptom

Missing Wi-Fi enums, macros, headers, or functions after the version bump.

#### Check Next

- ESP-IDF 6.0 Wi-Fi migration guide

#### Likely Fix Direction

- Replace removed enums or macros with their direct modern equivalents
- Update renamed APIs instead of creating local compatibility wrappers immediately
- Audit Wi-Fi event structure field changes before reusing old parsing logic

### `idf.py size --format json` Script Breakage

#### Symptom

CI or local tooling fails when parsing size output after the migration.

#### Check Next

- ESP-IDF 6.0 tools migration guide
- Build scripts or CI scripts that parse `idf.py size`

#### Likely Fix Direction

- Replace `idf.py size --format json` with `idf.py size --format json2`
- Update parsers for the newer hierarchical output structure

### `idf.py efuse*` Command Failures

#### Symptom

`idf.py efuse*` commands fail because no port was specified.

#### Check Next

- ESP-IDF 6.0 tools migration guide

#### Likely Fix Direction

- Pass `--port <PORT>` explicitly or set `ESPPORT`

### Environment Baseline Failures

#### Symptom

The migration cannot even reach application compile errors because tool execution fails first.

This can also show up as an early build warning that the project's declared CMake minimum is older than the expected ESP-IDF 6.0 baseline.

#### Check Next

- Python version
- CMake version
- `eim list`
- active `idf.py --version`

#### Likely Fix Direction

- Upgrade to a supported Python version
- Upgrade CMake to a supported version
- Check the top-level `CMakeLists.txt` and, if the build warns that the project minimum is too old, update it to `cmake_minimum_required(VERSION 3.22)`

## Version Coverage Summary

Use this section to decide which official migration chapters to open first. It is a routing aid, not a replacement for the real migration guides.

### 4.4 to 5.0

- Guide: https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.0/index.html
- Top-level coverage:
  - Bluetooth Classic
  - Bluetooth Low Energy
  - Build System
  - GCC
  - Networking
  - Peripherals
  - Protocols
  - Provisioning
  - Removed or Deprecated Components
  - Storage
  - System
  - Tools

### 5.0 to 5.1

- Guide: https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.1/index.html
- Top-level coverage:
  - GCC
  - Peripherals
  - Storage
  - Networking
  - System

### 5.1 to 5.2

- Guide: https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.2/index.html
- Top-level coverage:
  - GCC
  - Peripherals
  - Protocols
  - Storage
  - System
  - Wi-Fi
  - Bluetooth Classic

### 5.2 to 5.3

- Guide: https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.3/index.html
- Top-level coverage:
  - Bluetooth Classic
  - GCC
  - Peripherals
  - Protocols
  - Security
  - Storage
  - System

### 5.3 to 5.4

- Guide: https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.4/index.html
- Top-level coverage:
  - GCC
  - System
  - Bluetooth Classic
  - Bluetooth Common
  - Storage
  - Wi-Fi
  - Protocols

### 5.4 to 5.5

- Guide: https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-5.x/5.5/index.html
- Top-level coverage:
  - Build System
  - Security
  - System
  - Peripherals
  - Protocols
  - Wi-Fi

### 5.5 to 6.0

- Guide: https://docs.espressif.com/projects/esp-idf/en/latest/esp32/migration-guides/release-6.x/6.0/index.html
- Top-level coverage:
  - Bluetooth Classic
  - Build System
  - Networking
  - Peripherals
  - Provisioning
  - Protocols
  - Wi-Fi
  - Security
  - Tools
  - Storage
  - System
  - Toolchain
