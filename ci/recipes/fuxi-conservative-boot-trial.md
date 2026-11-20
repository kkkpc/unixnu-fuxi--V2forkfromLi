# fuxi conservative boot recipe trial

## Intent

This CI-only experiment asks whether a pinned GitHub Actions macOS/Xcode environment can build the clean committed board recipe. It does not establish that macOS environment differences caused previous boot failures. A successful build is not evidence of a successful device boot.

## Selected changes to `board.toml`

- Remove `fuxi-panic-fastboot=30`. That argument gates preparation of the reset backend, not the optional USB post-drain hook itself. On this branch the USB archive builder also omits the hook and rejects stale staged inputs containing it. This is a source-level safeguard, not a device test.
- Set `sheng-data-persist=1` for read-only inspection of the persistent Data-store window.
- Remove `sheng-data-reset=1`; the CI recipe must not request a reset of persistent Data.
- Set `sheng-gpu-power=7`, the board comment's read-only observation mode. Modes 8 and 9 reach the unqualified `gpuOn()` write sequence; the board history reports a device data abort at mode 9.
- Keep the remaining boot arguments and pinned board identity intact to limit the experiment to known risky controls.

This is a conservative boot-attempt hypothesis, not a proven highest-probability recipe. It retains the existing GPU/display stack and may still fail in firmware, storage, graphics, security, or unrelated paths.

## Build provenance

- Board repository base: `b4d1c207025195d769cb3232825691a84794b41d` (`V2`).
- GitHub workflow pins UniXNU `6ae4c252937ae1920ab68eaded10fc78447e0194`, UEFI `c5793bc2c60a2e2f5d9fbaad8717ed3f2736bfd2`, and requests Xcode `26.6` on `macos-26` (the workflow falls back to the newest installed Xcode 26).
- Workflow compiles the native kernel, checks selected linked/prelinked contracts, builds payload and DEBUG firmware, packs a boot image, and encrypts the image artifact to the repository's public recipient key.
- The payload step alone receives the board identity through its environment. No real value is used for host checks, written into the recipe or passed on the shell command line. If the required input is missing, the step must fail rather than package a substitute identity.
- Local control at `08927f1`: macOS 15.7.8 arm64, Xcode 26.3 (17C529), iPhoneOS SDK 26.2. The two touch tests passed under Apple clang ASan/UBSan; the board constants test also passed. The three C inputs from the Xcode workflow each produced an arm64 iOS object. These are not Linux runner results or a full kernel link.

## Verification boundary

Local TOML syntax and safe-token checks are host checks. The USB staging guard accepts the pinned clean source and rejects both kinds of known stale input (header macro or patched C function). The pinned NativeUfs write entry rejects mode 1; this is not a general no-writes guarantee for other drivers. The standalone porting-record checker is separate from the image job: two inherited audio host-evidence entries lack valid commit IDs. Keep that result separate from image-build status.

Before adding the private board identity on a fork, its owner must confirm that they control the private counterpart of the existing artifact recipient public key. A parseable public key is not evidence of ownership. Never submit a private key or identity in a PR or chat.

CI logs establish only the stages that actually completed under runner-provided tools. No device is enumerated, booted, flashed, reset, or recovered by this recipe. Any later trial boot and log collection remain owner-operated and need separate authorization.
