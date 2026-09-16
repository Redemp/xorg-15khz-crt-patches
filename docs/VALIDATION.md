# Validation and recorded results

## Initial patch provenance

I developed the initial seven patches and submitted them in [linux_kernel_15khz PR #18](https://github.com/D0023R/linux_kernel_15khz/pull/18) under my GitHub account, [Redemp](https://github.com/Redemp). Before publishing this standalone set, the patch context and line numbers were refreshed against the actual upstream release files. The intended source change remains `320, 200` to `192, 192` in `xf86CrtcSetSizeRange()`.

## Upstream source verification: 2026-09-16

All seven variants were checked against the complete target source file extracted from the corresponding official Xorg release archive:

| Source release | Context matches at the stated line | Patch applies | Only the MIN192 arguments change |
| --- | --- | --- | --- |
| Xorg Server 1.20.14 | Pass | Pass | Pass |
| Xorg Server 21.1.24 | Pass | Pass | Pass |
| xf86-video-amdgpu 19.1.0 | Pass | Pass | Pass |
| xf86-video-amdgpu 21.0.0 | Pass | Pass | Pass |
| xf86-video-amdgpu 22.0.0 | Pass | Pass | Pass |
| xf86-video-amdgpu 23.0.0 | Pass | Pass | Pass |
| xf86-video-amdgpu 25.0.0 | Pass | Pass | Pass |

The checks used `git apply --check --verbose` and actual application with Git's line-ending conversion disabled. Each original hunk matched the upstream file at its stated line, with no offsets. Comparing the complete resulting file with the original confirmed that only the two minimum-size arguments changed.

[source-verification.json](source-verification.json) records the source URLs, downloaded archive SHA-256 hashes, original source-file hashes, patch hashes, and individual results. These hashes identify the files used for this check; they are not a claim of separate signature verification.

Earlier checks used source-context fixtures. Fresh upstream verification found missing blank-line context in five variants and line offsets in two others. Those issues were corrected before generating the results above. The earlier fixture checks are superseded by this verification.

Runtime evidence is recorded for both Xorg driver paths: generic modesetting and dedicated AMDGPU, with xf86-video-amdgpu 25.0.0 named in the original patch library. The modesetting runtime result should not be read as proof that every Xorg Server 21.1 release was built or tested.

This source verification does not add a new distribution build or hardware test. No runtime result on Linux 7.2.6 or the Xorg Server 26 prerelease is claimed.

## Before and after

The saved Navi10/RDNA1 logs show the following configurations:

| Configuration | Connector output | Xorg screen framebuffer |
| --- | --- | --- |
| Stock minimum, `dotclock_min=0` | 256x224 | 320x224 |
| Stock minimum, `dotclock_min=6` | 512x224 | 512x224 |
| MIN192, native mode | 256x224 | 256x224 |

The stock native-mode log reports:

```text
Screen 0: minimum 320 x 200, current 320 x 224, maximum 16384 x 16384
DP-1 connected primary 256x224+0+0
```

The saved MIN192 causality test reports:

```text
visual=smooth
Screen 0: minimum 192 x 192, current 256 x 224, maximum 16384 x 16384
DP-1 connected primary 256x224+0+0
SR-1_256x224@60.10  60.10*
```

The visual result is a recorded test observation. The dimensions independently confirm that the patched Xorg screen could match the native output.

## Two related findings

### Navi10/RDNA1 native low-resolution judder

The history records the symptom on an AMD Radeon RX 5700 XT and reproduction with:

- 240p Test Suite for SNES.
- Sonic the Hedgehog 2 for Mega Drive/Genesis.
- Contra for NES.
- Super Mario Bros Remake, a native Linux application at 256x224/60 Hz.
- A Windows game through Wine at 256x224/60 Hz.

The Navi10 judder was also reported with Linux 6.18.16 and 7.1.6. This supports looking beyond a single emulator or kernel release. It does not by itself exclude every shared kernel/display interaction.

MIN192 removed the observed judder when Xorg could use the native framebuffer. The recorded hardware scope of this particular judder finding is Navi10/RDNA1.

### Broader AMD native-width/low-dotclock compatibility

The history and PR report black-screen or transition problems with narrow native modes across AMD APUs and both older and newer discrete GPUs. The Castlevania: Symphony of the Night transition problem with `dotclock_min=0` was reported to disappear with MIN192.

Increasing `dotclock_min` can select a horizontally multiplied mode, changing both width and pixel clock. That workaround alone does not establish a pixel-clock fault. The MIN192 change permits the narrow native Xorg framebuffer directly.

The evidence supports these recorded outcomes; it does not establish that all AMD GPUs exhibit the same judder or that MIN192 fixes every low-clock display problem.

## Scope

The patch changes Xorg's minimum screen-size range. Kernel 15 kHz mode support, pixel-clock restrictions, interlaced output, and direct KMS/KMSRAW behavior need their own validation. Applications can also make independent rendering-size decisions.

The exact lower-level presentation mechanism behind every symptom remains less certain than the observed benefit of allowing the native framebuffer.

## Evidence references

- [Original PR and testing update](https://github.com/D0023R/linux_kernel_15khz/pull/18).
- Maintainer's local history: `README_XORG_MIN192_PATCH_HISTORY.md`.
- Saved comparison logs: `NAVI10_DOTCLOCK_0.txt`, `NAVI10_DOTCLOCK_6.txt`, and the corresponding RetroArch launch logs.
- Saved runtime result: `NAVI10_XORG_MIN192_256_CAUSALITY_RESULT.zip`.

The local evidence filenames identify the original investigation artifacts. Those large or environment-specific artifacts are not included in this source-patch repository.
