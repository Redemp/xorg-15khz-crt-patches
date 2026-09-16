# Xorg 15 kHz CRT patches

Xorg patches for native low-resolution output on 15 kHz CRT displays, including the **MIN192** framebuffer fix for the modesetting and AMDGPU display drivers.

MIN192 lowers Xorg's minimum screen size from `320x200` to `192x192`, allowing the X screen framebuffer to match native CRT modes such as `256x224` and `256x240`.

The patch changes a userspace size restriction. A working CRT mode and the required kernel, GPU, adapter, and display support are still needed. Its effect is not restricted to 15 kHz displays.

## What MIN192 changes

Both supported Xorg display drivers set a minimum screen size with `xf86CrtcSetSizeRange()`. The change is:

```diff
-    xf86CrtcSetSizeRange(pScrn, 320, 200, mode_res->max_width,
+    xf86CrtcSetSizeRange(pScrn, 192, 192, mode_res->max_width,
                          mode_res->max_height);
```

`192x192` is a conservative tested minimum. It permits common native CRT resolutions while retaining a lower bound. It does not force the screen to run at `192x192`.

## Choose the correct patch

Select by the **Xorg display driver actually in use** and the **source version being built**.

| Loaded Xorg driver | Component to patch | Directory family |
| --- | --- | --- |
| `modesetting_drv.so` | Xorg Server's generic modesetting driver | `xserver-*` |
| `amdgpu_drv.so` | Dedicated AMDGPU Xorg driver | `xf86-video-amdgpu-*` |

Check the active Xorg log, for example:

```sh
grep -E 'modesetting_drv.so|amdgpu_drv.so' /var/log/Xorg.0.log
```

Some distributions store the log elsewhere. If both modules appear in the log, check the surrounding messages to identify the driver managing the intended screen. Check the installed package version using your distribution's package tools; the GPU model and kernel version do not select an Xorg patch.

The two components contain separate copies of this size restriction. A distribution shipping both drivers can patch both packages; an individual test must patch the driver that is active.

## Patch library

Version folders are at the repository root. Each contains a numbered patch, applied from the corresponding component's source directory with `patch -p1`.

| Directory | Exact source baseline | Recorded validation |
| --- | --- | --- |
| [xserver-1.20](xserver-1.20/) | Xorg Server 1.20.14 | Patch application verified against upstream source |
| [xserver-21.1](xserver-21.1/) | Xorg Server 21.1.24 | Patch application verified; MIN192 runtime evidence for the modesetting driver path |
| [xf86-video-amdgpu-19.1](xf86-video-amdgpu-19.1/) | xf86-video-amdgpu 19.1.0 | Patch application verified against upstream source |
| [xf86-video-amdgpu-21.0](xf86-video-amdgpu-21.0/) | xf86-video-amdgpu 21.0.0 | Patch application verified against upstream source |
| [xf86-video-amdgpu-22.0](xf86-video-amdgpu-22.0/) | xf86-video-amdgpu 22.0.0 | Patch application verified against upstream source |
| [xf86-video-amdgpu-23.0](xf86-video-amdgpu-23.0/) | xf86-video-amdgpu 23.0.0 | Patch application verified against upstream source |
| [xf86-video-amdgpu-25.0](xf86-video-amdgpu-25.0/) | xf86-video-amdgpu 25.0.0 | Patch application verified; runtime testing recorded for the AMDGPU driver path |

These are the seven variants from the initial investigation. Folder names identify version families; they do not promise that every release or distribution backport in that family has identical source context. Source verification and runtime testing are distinct; see [validation and test results](docs/VALIDATION.md).

Older variants remain available for systems that still use those components. Variants whose upkeep is discontinued can be moved to [unmaintained](unmaintained/).

### Release status

Checked against the official release archives on **2026-09-16**:

| Component | Latest listed stable release | Coverage |
| --- | --- | --- |
| [Xorg Server](https://www.x.org/releases/individual/xserver/) | 21.1.24 | Included; patch application verified |
| [xf86-video-amdgpu](https://www.x.org/releases/individual/driver/) | 25.0.0 | Included; patch application verified |

The Xorg Server archive also lists prerelease **26.0.99.902**. It is not covered by this initial set and needs a separate source review.

### Kernel compatibility

This library tracks **Xorg Server and xf86-video-amdgpu releases**. Its patches apply to those userspace components, so a new Linux kernel release does not by itself require a new MIN192 patch.

Linux **7.2.5** and **7.2.6** do not include an Xorg version. The distribution packages Xorg separately and can use the same Xorg packages with either kernel. For example, a system using Xorg Server 21.1.24 and the modesetting driver uses the existing `xserver-21.1/` patch with either kernel; a system using xf86-video-amdgpu 25.0.0 uses `xf86-video-amdgpu-25.0/`. No separate kernel-version MIN192 variant is needed for these combinations.

Kernel and hardware behavior still matter for the complete display setup. The recorded Navi10 runtime comparisons cover Linux **6.18.16** and **7.1.6**; they are not a claim of hardware testing on every newer kernel. Choose the patch by the Xorg source version and active driver, and test the resulting system on the kernel you intend to use.

## Apply and build

Obtain the source for the component and version your distribution uses. Apply the matching patch from inside that source directory.

For Xorg Server 21.1.24:

```sh
cd xorg-server-21.1.24
patch --dry-run --fuzz=0 -p1 < /path/to/xorg-15khz-crt-patches/xserver-21.1/01_xorg_low_resolution_min_192x192.patch
patch --fuzz=0 -p1 < /path/to/xorg-15khz-crt-patches/xserver-21.1/01_xorg_low_resolution_min_192x192.patch
```

For xf86-video-amdgpu 25.0.0:

```sh
cd xf86-video-amdgpu-25.0.0
patch --dry-run --fuzz=0 -p1 < /path/to/xorg-15khz-crt-patches/xf86-video-amdgpu-25.0/01_amdgpu_low_resolution_min_192x192.patch
patch --fuzz=0 -p1 < /path/to/xorg-15khz-crt-patches/xf86-video-amdgpu-25.0/01_amdgpu_low_resolution_min_192x192.patch
```

Review the dry-run result before applying. Investigate failed hunks or offsets against the exact source package. The examples use GNU patch.

Build, package, and install the changed component using the normal procedure for your distribution, then restart Xorg or reboot. This repository supplies source patches; the build and installation procedure depends on the distribution.

## Verify the result

In the Xorg session, run:

```sh
xrandr | head -n 1
```

The minimum should now read `192 x 192`. After selecting an available native `256x224` mode with your existing CRT/Switchres setup, the screen size should also match:

```text
Screen 0: minimum 192 x 192, current 256 x 224, ...
```

Check both the overall `Screen 0` dimensions and the connector's active mode. A connector using `256x224` alone does not prove the Xorg framebuffer is also `256x224`.

## Why this exists

The original investigation found that a CRT could receive `256x224` while Xorg retained a `320x224` framebuffer because of its minimum-width restriction. The MIN192 test allowed a matching `256x224` framebuffer, with smooth presentation recorded on the Navi10/RDNA1 test system.

The testing history distinguishes two findings:

- **Native low-resolution judder:** reproduced on Navi10/RDNA1 and removed by MIN192 in the tested configuration. Reproduction included RetroArch workloads, a native Linux game, and a Windows game through Wine.
- **Native-width/low-dotclock compatibility:** black-screen or transition problems were reported across several AMD generations. The recorded Castlevania: Symphony of the Night case with `dotclock_min=0` disappeared with MIN192. Wider, horizontally multiplied modes had previously masked the issue.

The detailed evidence and its limits are in [docs/VALIDATION.md](docs/VALIDATION.md). MIN192 does not change kernel mode validation, hardware pixel-clock limits, interlace support, or direct KMS/KMSRAW operation.

## Maintenance and origin

I developed these patches to address Xorg's minimum framebuffer-size restriction for native low-resolution CRT output, and I maintain them here. I go by **Rion**, or [Redemp](https://github.com/Redemp) on GitHub.

I originally submitted the patch set in [linux_kernel_15khz PR #18](https://github.com/D0023R/linux_kernel_15khz/pull/18). The restriction applies across Linux distributions using the affected, unpatched Xorg drivers.

The version-folder layout follows [linux_kernel_15khz](https://github.com/D0023R/linux_kernel_15khz). Here the folders follow Xorg component versions.

See [CONTRIBUTING.md](CONTRIBUTING.md) for updating patches and reporting test results.

Upstream sources:

- [Xorg Server release archives](https://www.x.org/releases/individual/xserver/)
- [Xorg display-driver release archives](https://www.x.org/releases/individual/driver/)

## License

My original patch contributions and documentation are licensed under the [MIT License](LICENSE), copyright 2026 Rion (Redemp).

Upstream source context retains its original attribution and permission terms; see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
