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
| `frameworks/native` | [damnms/android_frameworks_native](https://github.com/damnms/android_frameworks_native) | `lineage-18.1-shieldtablet` |
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
  the HWC1 GL composer runs with one RPC thread, the HWC1 adapter closes
  the framebuffer target fence (fd leak, crash on HDMI hotplug).
- HDMI: the HWC1 adapter prepares and sets one display at a time (as
  SurfaceFlinger presents them) and reports a monitor connected at boot;
  the kernel announces HDMI audio after the HDA setup.
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
- USB tethering and the Wi-Fi hotspot: the kernel has the `rpfilter`
  match netd needs for tethering; hostapd may enter `/data/vendor/wifi`;
  the Wi-Fi HAL switches bcmdhd between station and SoftAP firmware mode
  (`op_mode`), so the hotspot sends beacons; the hotspot can use 5 GHz.
  The driver refuses the p2p device in SoftAP mode instead of crashing.
  Without mobile data the hotspot has no internet source (the chip runs
  either station or SoftAP, not both).
- GPS (Broadcom BCM4752): the daemon and HAL module from the 15.1/16.0
  vendor tree are back, with the GNSS 1.0 HAL as a binderized service, a
  shim for `SSLv3_client_method` (gone from BoringSSL) and SELinux rules.
  It works standalone: no daily LTO download and no RTI status download
  from Broadcom, no PSDS servers, no SUPL. The GNSS wrapper copies the
  NMEA sentences of the Broadcom module (they are not null-terminated),
  and the system init creates `/data/system/gps`, so GPS also starts
  after *Format data*.
- Audio: a 4-pole headset's microphone can be used for recording (the
  capture route left out the wired headset mic).
- Screen recording from Quick Settings: SystemUI no longer forces AVC
  Main level 3 (too small for the screen, the NVIDIA encoder hung), and
  the kernel no longer asks for the `open` permission on `ftruncate()`
  of an already open file (backport of upstream "selinux: do not check
  open perm on ftruncate call"), so mediaserver can finish the file.
- Wi-Fi reconnects within seconds with the screen off: hardware PNO is
  enabled in the Wi-Fi overlay (Android 11 ignored the old setting).
- Boot without dead setup steps: no cpuset writes (the 3.10 kernel has
  no cpusets), no unused `/mnt/tmp`, no touch `.ini` links or NvCPL
  keylayout directories in `/data`; fstrim no longer logs a denial for
  the calibration partition.
- Wi-Fi networks the chip cannot join are shown in the Wi-Fi list as
  "Not supported by this device (WPA3 or PMF required)" and cannot be
  selected: WPA3 only, or only the SHA256 suites (WPA2 with PMF required).
  Quick Settings open the Wi-Fi list for them instead of the password page.
- Wi-Fi with "Use device MAC" connects with the factory MAC; the driver
  reported the placeholder `00:90:4c:11:22:33` as the permanent address.
- The backlight stays at PWM level 30 or above (the panel flickers below):
  slider minimum, the dimmed screen before screen off and the darkest
  automatic brightness level.
- Wi-Fi Direct: the tablet is visible and connects also with a randomized
  MAC (the P2P device address is derived from the factory MAC); switching
  Wi-Fi off with the Wi-Fi Direct page open no longer panics the kernel.
- Chrome/Chromium no longer shows garbled content: the NVIDIA HWC1 shows
  buffers of SurfaceControl layers wrongly as overlays, so SurfaceFlinger
  composes these layers on the GPU (`debug.sf.client_composition_buffer_state`).
- Encryption ("Encrypt tablet") and the screen lock (PIN, pattern,
  password): the software keymaster and gatekeeper replace NVIDIA's TLK
  keymaster, which cannot create keys without `tlk_daemon` (its storage is
  on `/data`); vold looped on "Failed to generate keypair".
- Boot with an encrypted `/data`: credstore, tombstoned, nvphsd, the GNSS
  HAL and gpsd are stopped while vold switches from the temporary to the
  decrypted `/data` (`class_reset_post_data` does nothing without
  updatable APEX), so the unmount no longer fails with EBUSY.

Known and harmless: a few denials of platform domains (`storaged` reading
eMMC statistics, Settings calling `storaged`/`wificond`, Gallery reading a
camera property) and boot log lines from AOSP or configfs init files that
do not apply to this kernel. None of them has a visible effect.

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
- Wi-Fi security: WPA2-PSK, WPA2-Enterprise and WPA2/WPA3 transition
  networks work. The bcm43241 firmware has no SAE and no PMF (802.11w), so
  WPA3-only networks and networks with PMF required cannot work.
- Wi-Fi Direct runs on the same channel as the Wi-Fi connection (the
  firmware has no multi-channel support). While connected to a network on
  a 5 GHz DFS channel (52–144), Wi-Fi Direct connections fail; they work
  without a Wi-Fi connection or with a network on 2.4 GHz or channels
  36–48.
- `vendor/nvidia` contains NVIDIA's proprietary binaries as published by
  TheMuppets; three camera libraries and `libglcore` carry small symbol
  renames (see `device/nvidia/t124-common/extract/patches.sh`).
