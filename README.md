# Prebuilt kernel for Redmi Pad Pro (dizi)

Everything here is taken unmodified from dizi_eea OS3.0.303.0.WNSEUXM:

| File | Source |
|---|---|
| `Image` | `boot.img` kernel (GKI 5.10.246-android12-9, ab15159037) |
| `dtb/parrot.dtb` | `vendor_boot.img` dtb |
| `dtbo.img` | `dtbo.img` |
| `modules/vendor_ramdisk/` | `vendor_boot.img` ramdisk `lib/modules` |
| `modules/vendor_dlkm/` | `vendor_dlkm` partition `lib/modules` |

## Source-built kernel (Phase 6, stage a)

`Image-source` is built from LineageOS android_kernel_xiaomi_sm7435 lineage-23.2
(57dcedf23, plus the local fix a67b9494c
"proc: bootconfig: Keep the skip label inside its #ifdef") with plain `gki_defconfig` and the
stock kernel's compiler, clang r416183b (`/build/alex/dizi/kernel/build-gki.sh`):
Linux 5.10.269. Its Module.symvers matches every symbol CRC, and `module_layout`, that the 377
stock modules import, so the stock dtb, dtbo and modules above are used unchanged.
Select it with `DIZI_SOURCE_KERNEL=true` (see BoardConfig.mk). The stock `Image` is the default.

## Source-built display driver (Phase 6, stage b, experimental)

`modules-source/{baseline,splashfix}/msm_drm.ko` are built from MiCode's
vendor_opensource_display-drivers (ruan-u-oss) against the source kernel
(`/build/alex/dizi/kernel/out-display`, research/kernel-stage-b-display.md). All 823 imports and
51 exports match the stock module's CRCs, and no stock module imports from msm_drm. `splashfix` adds
kernel/patches/0002 (reprogram the INTF/DSI timing at the continuous-splash handoff when the
bootloader left another refresh rate). Select one with `DIZI_SOURCE_DISPLAY=baseline|splashfix`
(replaces msm_drm.ko in both vendor_ramdisk and vendor_dlkm).

## ax_dragonite.ko (AxionOS)

`modules/vendor_dlkm/ax_dragonite.ko` is AxionOS's thread affinity/boost helper
(device/axion/common/kernel/modules/ax_dragonite, lineage-23.2 d80f522), built
against kernel_xiaomi_ruan (gki_defconfig + vendor/ruan_rom.config, clang-r416183b)
and stripped of debug info. All 42 imports match that kernel's CRCs. It is not
in modules.load: AxionOS's init.axion.modules.rc modprobes it at early-init.

Lineage's kernel.mk cannot build it in the ROM build here: it only builds
external modules alongside in-tree ones, and then writes modules.dep/modules.load
for vendor_dlkm from the source-built modules alone, over the prebuilt ones.

Rebuild it whenever the kernel's KMI changes (e.g. after an upstream merge):

    make -C <kernel> O=<out> ARCH=arm64 LLVM=1 LLVM_IAS=1 \
        CROSS_COMPILE=aarch64-linux-gnu- M=<ax_dragonite dir> modules
    llvm-strip --strip-debug -o modules/vendor_dlkm/ax_dragonite.ko <built .ko>

With the stock Image (RUAN_PREBUILT_KERNEL=true) it does not load (different
CRCs); nothing else depends on it.
