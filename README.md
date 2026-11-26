# Local manifest for the NVIDIA SHIELD Tablet (shieldtablet), LineageOS 20

This local manifest builds LineageOS 20 (Android 13) for the NVIDIA
SHIELD Tablet K1 / SHIELD Tablet, codename `shieldtablet` (Tegra K1, T124).
LineageOS never shipped 20 for this device. This branch is a bring-up:
it boots and runs daily use, there is no release yet. LineageOS 18.1 for
this device is on branch `lineage-18.1`.

Android 12 and later need eBPF in the kernel; the 3.10 kernel here carries
a backport (eBPF, cgroup v2, cpusets, file-based encryption, a kernel
version spoof for bpfloader and system_server).

## Repositories

| Path | Repository | Revision |
| --- | --- | --- |
| `device/nvidia/shieldtablet` | [damnms/android_device_nvidia_shieldtablet](https://github.com/damnms/android_device_nvidia_shieldtablet) | `lineage-20.0-shieldtablet` |
| `device/nvidia/t124-common` | [damnms/android_device_nvidia_t124-common](https://github.com/damnms/android_device_nvidia_t124-common) | `lineage-20.0-shieldtablet` |
| `device/nvidia/tegra-common` | [damnms/android_device_nvidia_tegra-common](https://github.com/damnms/android_device_nvidia_tegra-common) | `lineage-20.0-shieldtablet` |
| `device/nvidia/shield-common` | [damnms/android_device_nvidia_shield-common](https://github.com/damnms/android_device_nvidia_shield-common) | `lineage-20.0-shieldtablet` |
| `device/nvidia/sepolicy` | [damnms/android_device_nvidia_sepolicy](https://github.com/damnms/android_device_nvidia_sepolicy) | `lineage-20.0-shieldtablet` |
| `device/nvidia/icera` | [damnms/android_device_nvidia_icera](https://github.com/damnms/android_device_nvidia_icera) | `lineage-20.0-shieldtablet` |
| `device/nvidia/touch` | [damnms/android_device_nvidia_touch](https://github.com/damnms/android_device_nvidia_touch) | `lineage-18.1-shieldtablet` |
| `kernel/nvidia/shield` | [damnms/android_kernel_nvidia_shield](https://github.com/damnms/android_kernel_nvidia_shield) | `lineage-20.0-shieldtablet` |
| `vendor/nvidia` | [damnms/proprietary_vendor_nvidia](https://github.com/damnms/proprietary_vendor_nvidia) | `lineage-20.0-shieldtablet` |
| `frameworks/av` | [damnms/android_frameworks_av](https://github.com/damnms/android_frameworks_av) | `lineage-20.0-shieldtablet` |
| `frameworks/base` | [damnms/android_frameworks_base](https://github.com/damnms/android_frameworks_base) | `lineage-20.0-shieldtablet` |
| `frameworks/native` | [damnms/android_frameworks_native](https://github.com/damnms/android_frameworks_native) | `lineage-20.0-shieldtablet` |
| `hardware/interfaces` | [damnms/android_hardware_interfaces](https://github.com/damnms/android_hardware_interfaces) | `lineage-20.0-shieldtablet` |
| `packages/apps/Jelly` | [damnms/android_packages_apps_Jelly](https://github.com/damnms/android_packages_apps_Jelly) | `lineage-20.0-shieldtablet` |
| `system/memory/lmkd` | [damnms/android_system_memory_lmkd](https://github.com/damnms/android_system_memory_lmkd) | `lineage-20.0-shieldtablet` |
| `hardware/nvidia/*` | LineageOS, unchanged | `lineage-20.0` |

The device trees, kernel and blobs continue the 18.1 forks; the shared
NVIDIA repositories and the framework forks start from LineageOS 20
(`lineage-20` / `lineage-20.0`). The rest of the tree comes from the
LineageOS manifest, branch `lineage-20.0`.

## What works

Display and GPU (NVIDIA GLES, HWC1), Wi-Fi, Bluetooth, audio playback and
recording, camera (photo and video), hardware video decoding and encoding,
sensors, GPS daemon, USB adb, SELinux enforcing.

## Android 13 specifics on this device

- The kernel gained FunctionFS AIO (adbd), `IFA_FLAGS` in address
  notifications (DHCP in the network stack) and a fixed touchscreen read.
- The CPU tuning is `cortex-a7` with `cortex-a15` at runtime: Clang 14
  miscompiles the Brotli decoder for `cortex-a15` (OTA install failed).
- ueventd reads the NVIDIA rules from `/vendor/etc/ueventd.rc`; partitions,
  NVIDIA properties and the camera's JPEG encoder are labeled for the
  Android 13 SELinux policy.
- Without VNDK, HIDL and `libbinder_ndk` share one process state; the
  audio service stays off vndbinder.
- The boot animation frees each part's textures once it is done (it kept
  up to 879 MiB in GPU memory and ran the 2 GB device out of memory).
- Camera: cameraserver skips its JPEG header fixup
  (`ro.vendor.camera.skip_hidl_jpeg_blob_fixup`, the NVIDIA HAL still holds
  the buffer lock) and ImageReader reads the unconverted header; ACodec
  derives the AVC level when an app sets only a profile (CameraX).
- nvphsd runs without the `il` governor: it reads `/proc/<pid>/largest_task`,
  which the 3.10 kernel lacks, and never changed a clock limit.
- The 18.1 fixes for HWC1, camera fences, audio policy, ACodec, lmkd,
  screen recording and Jelly are carried over.

Every change is a separate commit (Conventional Commits) with the reason
in its message.

## Usage

The tree builds in the same Ubuntu 20.04 container as 18.1.

```bash
mkdir -p ~/android/lineage-20.0 && cd ~/android/lineage-20.0
repo init -u https://github.com/LineageOS/android.git -b lineage-20.0 --git-lfs --depth=1
git clone -b lineage-20.0 https://github.com/damnms/android_local_manifests_shieldtablet.git .repo/local_manifests
repo sync -c -j4 --no-tags --no-clone-bundle
```

Build:

```bash
source build/envsetup.sh
lunch lineage_shieldtablet-userdebug
m bacon -j16
```

The result is
`out/target/product/shieldtablet/lineage-20.0-<date>-UNOFFICIAL-shieldtablet.zip`.

## Notes

- The zip writes only system and boot. Coming from 18.1 or another ROM
  needs the recovery from this build (`fastboot flash recovery`) and
  *Format data*.
- Release builds are signed with private keys that are not part of any
  repository; without your own keys the build uses the public test keys.
- Open points: headphones and HDMI not tested yet, the Wi-Fi hotspot
  offload (BPF maps cannot be labeled on 3.10).
