# Maintaining the patches

## Directory and filename conventions

- `xserver-X.Y/`: patches for the modesetting driver inside Xorg Server.
- `xf86-video-amdgpu-X.Y/`: patches for the dedicated AMDGPU Xorg driver.
- `unmaintained/`: previously checked variants whose ongoing upkeep has ended.

Keep the existing MIN192 filenames:

```text
01_xorg_low_resolution_min_192x192.patch
01_amdgpu_low_resolution_min_192x192.patch
```

Keep the classic `Index:` and `.orig/` unified-diff format, with paths suitable for `patch -p1`. Preserve Linux line endings. Number any additional patches in application order within the same component folder.

## Updating a version

1. Obtain the exact tagged release or distribution source package.
2. Check whether `xf86CrtcSetSizeRange()` still imposes the same minimum. If upstream has removed the restriction, document that outcome rather than carrying an unnecessary patch.
3. Check the source file and surrounding code. The established locations are `hw/xfree86/drivers/modesetting/drmmode_display.c` in Xorg Server and `src/drmmode_display.c` in xf86-video-amdgpu.
4. Dry-run the existing patch against a clean source tree with `patch --dry-run --fuzz=0 -p1`. Inspect all output, including offsets.
5. If the existing hunk still applies cleanly, record the additional exact source version. Update the hunk or add a version-specific variant when source context requires it.
6. Apply it and inspect the complete source diff. MIN192 should change only the two minimum-size arguments, from `320, 200` to `192, 192`.
7. Build the distribution package where possible. Test the installed driver on hardware where available.
8. Update the README compatibility table and `docs/VALIDATION.md` with the exact versions, date, method, and result. State separately whether the work was source-checked, patch-application checked, built, or runtime-tested.

Do not infer support for an untested release just from its version-family name. A source-context fixture check validates a patch's formatting and expected hunk; it is not a full-source build or hardware test.

## Release tracking

Check the official [Xorg Server](https://www.x.org/releases/individual/xserver/) and [display-driver](https://www.x.org/releases/individual/driver/) release listings when updating this library.

At initial repository preparation on 2026-09-16, the listings included the existing stable baselines Xorg Server 21.1.24 and xf86-video-amdgpu 25.0.0. They also listed the newer Xorg Server prerelease 26.0.99.902. That prerelease has no verified patch in this initial set; inspect its source before adding a variant. Experimental variants can be placed under `staging/` when there is an actual patch to evaluate.

Move a version to `unmaintained/` when this project's maintenance coverage for it ends, and record the reason and last checked baseline. The initial root-level versions describe available patch variants, not upstream support promises.

## Reporting a problem or a successful test

Include:

- Distribution, kernel version, GPU, and display/adapter setup.
- Active Xorg driver and exact package version, including downstream patches if known.
- Patch file used, its repository commit, and the patch/build output.
- The first line of `xrandr` and the connector's active mode before and after patching.
- Application and test case, modeline, refresh rate, and relevant Switchres settings such as `dotclock_min`.
- Separate observations for judder, tearing, black screens, and mode transitions.

Use the same test settings for the unpatched and patched comparison. Keep the Navi10/RDNA1 judder finding separate from the broader AMD native-width/low-dotclock findings.
