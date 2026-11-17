# fuxi conservative boot recipe trial

## Intent

This CI-only experiment asks whether a pinned GitHub Actions macOS/Xcode environment can build the clean committed board recipe. It does not establish that macOS environment differences caused previous boot failures. A successful build is not evidence of a successful device boot.

## Selected changes to `board.toml`

- Remove `fuxi-panic-fastboot=30` so a panic does not request automatic fastboot recovery.
- Set `sheng-data-persist=1` for read-only inspection of the persistent Data-store window.
- Remove `sheng-data-reset=1`; the CI recipe must not request a reset of persistent Data.
- Set `sheng-gpu-power=7`, the board comment's read-only observation mode. Modes 8 and 9 reach the unqualified `gpuOn()` write sequence; the board history reports a device data abort at mode 9.
- Keep the remaining boot arguments and pinned board identity intact to limit the experiment to known risky controls.

This is a conservative boot-attempt hypothesis, not a proven highest-probability recipe. It retains the existing GPU/display stack and may still fail in firmware, storage, graphics, security, or unrelated paths.

## Build provenance

- Board repository base: `b4d1c207025195d769cb3232825691a84794b41d` (`V2`).
- GitHub workflow pins UniXNU `6ae4c252937ae1920ab68eaded10fc78447e0194`, UEFI `c5793bc2c60a2e2f5d9fbaad8717ed3f2736bfd2`, and requests Xcode `26.6` on `macos-26` (the workflow falls back to the newest installed Xcode 26).
- Workflow compiles the native kernel, checks selected linked/prelinked contracts, builds payload and DEBUG firmware, packs a boot image, and encrypts the image artifact to the repository's public recipient key.
- At recipe preparation, no active Actions runs were listed. Actions are enabled. Repository-level Actions secret names were empty; the build may therefore stop at payload creation if no applicable organization secret supplies `UNIXNU_UNIQUE_CHIP_ID`. Never print or commit that seed.

## Verification boundary

Local TOML syntax and safe-token checks are host checks. CI logs establish only the stages that actually completed under runner-provided tools. No device is enumerated, booted, flashed, reset, or recovered by this recipe. Any later trial boot and log collection remain owner-operated and need separate authorization.
