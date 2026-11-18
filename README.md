# Local manifest for the NVIDIA SHIELD Tablet (shieldtablet), LineageOS 18.1

This local manifest builds LineageOS 18.1 (Android 11) for the NVIDIA
SHIELD Tablet K1 / SHIELD Tablet, codename `shieldtablet` (Tegra K1, T124).
LineageOS never shipped 18.1 for this device; 18.1 is also the last
version the 3.10 kernel can run (Android 12 needs eBPF).

## Repositories

| Path | Repository | Revision |
| --- | --- | --- |
| `device/nvidia/shieldtablet` | [damnms/android_device_nvidia_shieldtablet](https://github.com/damnms/android_device_nvidia_shieldtablet) | `lineage-18.1-shieldtablet` |
| `device/nvidia/t124-common` | [damnms/android_device_nvidia_t124-common](https://github.com/damnms/android_device_nvidia_t124-common) | `lineage-18.1-shieldtablet` |
| `device/nvidia/tegra-common` | [damnms/android_device_nvidia_tegra-common](https://github.com/damnms/android_device_nvidia_tegra-common) | `lineage-18.1-shieldtablet` |
| `device/nvidia/shield-common` | [damnms/android_device_nvidia_shield-common](https://github.com/damnms/android_device_nvidia_shield-common) | `lineage-18.1-shieldtablet` |
| `device/nvidia/touch` | [damnms/android_device_nvidia_touch](https://github.com/damnms/android_device_nvidia_touch) | `lineage-18.1-shieldtablet` |
| `device/nvidia/sepolicy` | [damnms/android_device_nvidia_sepolicy](https://github.com/damnms/android_device_nvidia_sepolicy) | `lineage-18.1-shieldtablet` |
| `kernel/nvidia/shield` | [damnms/android_kernel_nvidia_shield](https://github.com/damnms/android_kernel_nvidia_shield) | `lineage-18.1-shieldtablet` |
| `vendor/nvidia` | [damnms/proprietary_vendor_nvidia](https://github.com/damnms/proprietary_vendor_nvidia) | `lineage-18.1-shieldtablet` |
| `frameworks/av` | [damnms/android_frameworks_av](https://github.com/damnms/android_frameworks_av) | `lineage-18.1-shieldtablet` |
| `frameworks/base` | [damnms/android_frameworks_base](https://github.com/damnms/android_frameworks_base) | `lineage-18.1-shieldtablet` |
| `frameworks/opt/net/wifi` | [damnms/android_frameworks_opt_net_wifi](https://github.com/damnms/android_frameworks_opt_net_wifi) | `lineage-18.1-shieldtablet` |
| `hardware/interfaces` | [damnms/android_hardware_interfaces](https://github.com/damnms/android_hardware_interfaces) | `lineage-18.1-shieldtablet` |
| `system/memory/lmkd` | [damnms/android_system_memory_lmkd](https://github.com/damnms/android_system_memory_lmkd) | `lineage-18.1-shieldtablet` |
| `device/nvidia/icera`, `hardware/nvidia/*` | LineageOS, unchanged | `lineage-18.1` |

All forks keep their upstream branch `lineage-18.1` untouched; the device
work sits on top of it on `lineage-18.1-shieldtablet`. The rest of the tree
comes from the LineageOS manifest, branch `lineage-18.1`.

## What was fixed

- Boot with SELinux enforcing: a T124 policy (`t124-common/sepolicy`),
  watchdogd in recovery, the usercalib partition.
- Display: largest `vmalloc=` wins (the bootloader appends a smaller one),
  the HWC1 GL composer runs with one RPC thread.
- Binder: the 3.10 driver stored the opening thread instead of the process;
  fd transfers to apps failed once that thread exited (hardware bitmaps,
  ashmem, camera buffers). Backport of upstream "binder: use group leader
  instead of open thread".
- Hardware video decoding and encoding (NVIDIA OMX): shim for
  `libnvmmlite_video`, ACodec tolerates input port events.
- Camera (rear and front): EGL device emulation for the camera stack,
  SELinux rules, PHS hints skipped in apps, acquire fence leak fixed.
- Wi-Fi: 5 GHz DFS channels and WPA2/WPA3 transition networks; the country
  code is taken from nearby access points (802.11d), unknown countries stay
  on the worldwide locale.
- Memory: lmkd parses `/proc/zoneinfo` of pre-4.8 kernels again.
- WebView/browser: no GL preload in zygote; libglcore realloc fix (the
  browser no longer waits ~10 s on start).
- Sensors as a binderized HAL, own OTA and changelog URLs.
- Wi-Fi uses a randomized MAC per network; scans skip channels the driver
  does not offer yet (no stall on DFS networks after boot).
- The NVIDIA Games (TegraZone) app is left out; it cannot be updated and
  ran telemetry in the background.
- Recovery: *Format data* no longer ends with "Data wipe failed" (the
  fstab listed the crypto footer partition as an `emmc` `/metadata`
  volume).
- The kernel version string carries a fixed build user and host.

Every change is a separate commit (Conventional Commits) with the reason
in its message.

## Usage

The 18.1 tree needs an older userland (Python 2, libncurses5); building in
an Ubuntu 20.04 container works.

```bash
mkdir -p ~/android/lineage-18.1 && cd ~/android/lineage-18.1
repo init -u https://github.com/LineageOS/android.git -b lineage-18.1 --git-lfs --depth=1
git clone -b lineage-18.1 https://github.com/damnms/android_local_manifests_shieldtablet.git .repo/local_manifests
repo sync -c -j8 --no-tags --no-clone-bundle
```

Build:

```bash
source build/envsetup.sh
lunch lineage_shieldtablet-userdebug
m bacon -j16
```

Use `lunch` rather than `breakfast`: `breakfast` would add the LineageOS
device repositories a second time via roomservice. The result is
`out/target/product/shieldtablet/lineage-18.1-<date>-UNOFFICIAL-shieldtablet.zip`.

## Notes

- Release builds are signed with private keys that are not part of any
  repository. Without your own keys in `vendor/lineage-priv/keys`, the
  build is signed with the public test keys.
- The zip checks for bootloader `4.00.2016.04` and writes only system and
  boot. Switching from another ROM needs the recovery from this build
  (`fastboot flash recovery`) and *Format data*.
- From the LineageOS recovery, reboot with *Reboot system now*;
  `adb reboot` makes the bootloader load the recovery again.
- `vendor/nvidia` contains NVIDIA's proprietary binaries as published by
  TheMuppets; three camera libraries and `libglcore` carry small symbol
  renames (see `device/nvidia/t124-common/extract/patches.sh`).
