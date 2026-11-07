# Device: Qualcomm Technologies, Inc. kona MTP 19811 (oneplus8pro)

Made by `tools/new-device.py` on 2026-11-08, from the file fdt.dtb. Edit it as the port goes.

| What | Value |
|---|---|
| Board (the tree's `model`) | Qualcomm Technologies, Inc. kona MTP 19811 |
| Compatible | `qcom,kona-mtp qcom,kona qcom,mtp` |

## The SoC addresses

No reference tree to compare with (devices/redwood/prebuilt/device-tree.dtb is missing).

| Argument | Value | Against the reference |
|---|---|---|
| `soc_ipa_base` | `0x1e40000` |  |
| `soc_gsi_base` | `0x1e04000` |  |
| `soc_ipcc` | `0x408000` |  |
| `soc_aoss` | `0xc300000` |  |
| `soc_rsc` | `0x18220000` |  |
| `soc_apps_smmu` | `0x15000000` |  |
| `soc_cmd_db` | `0x80860000` | filled by hand: kona names the node `reserved-memory@80860000` (`qcom,cmd-db`), which soc-args.py's `cmd_db` leaf match misses; the address equals the reference's |
| `soc_mpss_mem` | `(not found)` | **missing, and truly absent**: the tree has no mpss reserved-memory node at all; fill from the phone at bring-up |
| `modem_ipa_fw_at` | `0x8e210000` | filled by hand: kona names the ipa_gsi reservation `pil_ipa_gsi_region@8e210000`, which soc-args.py's exact `ipa_gsi` leaf match misses |
| `soc_smem` | `0x80900000` |  |
| `soc_imem` | `0x14600000` |  |
| `soc_ipa_stream` | `0x5c0` |  |
| `soc_ipa_stream_uc` | `0x5c2` |  |
| `soc_pas_mpss` | `(not found)` | **missing, and truly absent**: the tree carries `qcom,pas-id` only for the non-modem peripherals (no `qcom,mss` node); fill from the phone at bring-up |
| `soc_pas_ipa` | `0xf` |  |
| `soc_ipa_fw` | `ipa_fws` |  |

`soc.args` is what `tools/soc-args.py` read out of the tree; `pull-soc.sh` makes it again from the phone on
adb. The run scripts add it to the kernel command line. Where each argument comes from in the tree:
`devices/redwood/README.md`.

## What is this phone's own

The kernel, loader and user space know nothing about a phone: addresses, interrupts, rails and firmware
come from its tree and its Android. What is written for one phone is the list below. The head of each file
says where its numbers came from; that is the map of what to rewrite.

- [ ] **UEFI**: the firmware that starts the loader: devices/redwood/uefi/ is a port of Mu-Silicium; the vendor's drivers are extracted from your own phone (uefi/README.md)
- [ ] **Display, buttons, touch**: drivers/input/hakios.input.c, hakios.buttons.c, hakios.goodix.c (the reference phone's controller; another phone has another)
- [ ] **Storage**: hakios.ufs.c and hakios.ufshc.c (UFS); a phone with eMMC needs another driver
- [ ] **CPU clocks**: hakios.cpufreq.c
- [ ] **GPU**: hakios.adreno.c (Adreno 642L here: another generation needs its own firmware and registers)
- [ ] **Wi-Fi**: hakios.wifi.c, hakios.lx.c and wifi/lx/ (the WCN6750 radio of the reference phone)
- [ ] **Audio and battery**: hakios.adsp.c, hakios.audio.c, hakios.sound.c, hakios.speaker.c, hakios.mic.c, hakios.battery.c
- [ ] **Cellular**: hakios.mpss.c, hakios.qmi.c, hakios.nas.c, hakios.ipa.c, hakios.cell.c; soc.args carries the addresses, the IPA layout is for IPA 4.11 (docs/cellular.md)
- [ ] **Haptics and cameras**: hakios.haptic.c, hakios.camera.c

## Configuration

The kernel's settings are layered the way an Android kernel's are: the common kernel, the SoC's, this phone's
(`docs/configs.md`). Two short files make this phone a configuration:

    configs/oneplus8pro.config          CONFIG_DEVICE="oneplus8pro"   and whatever is true of this phone only
    configs/oneplus8pro_defconfig       #include common_defconfig
                                   #include <the SoC's fragment: sm8250.config for the SM8250 family>
                                   #include oneplus8pro.config

Then `make oneplus8pro_defconfig`. Turn on only what this phone has been shown to do; `tools/hakios-config.py check` says what the
choice needs.

The drivers are part of the configuration too (`docs/drivers-config.md`): every one is built unless a configuration leaves it out,
and the addresses, pins and rails in each are the reference phone's. Leave out in `configs/oneplus8pro.config` what has not been
ported to this phone yet (`# CONFIG_HAKIOS_GOODIX is not set`, ...; `configs/qemu_defconfig` leaves out all of them), and put
each one back when it has been made to work here. `tools/hakios-config.py show` lists what is built in.

## Firmware

Pulled from your own phone, never committed: `devices/redwood/pull-firmware.sh`, or
`pull-firmware-root.sh` where the firmware is readable by root only. Check with `make doctor`.

## Before you open a pull request

`tools/provenance-check.py` must pass: no vendor firmware, no UEFI drivers, no Apple files in the tree.

## The repository

A HakiOS device repository: what is true of this device and of no other. The kernel, the loader, the drivers and the tools are
the common repository's; the sm8250 family's facts are in `soc/`, carried with the device. How the three are put together:
[docs/porting/repositories.md](https://github.com/oosexgang/hakios-xnu-kernel/blob/main/docs/porting/repositories.md).

| File | What |
|---|---|
| `metal.args` | what `tools/metal-run.sh` gives this phone: its cores, its panel's rate, the threads that copy frames, the touch poll |
| `device.conf` | what the device is, and how far each part of it works: its page in the documentation is made from it |
| `configs/oneplus8pro_defconfig` | the common kernel, the SoC's fragment, this device's: `make oneplus8pro_defconfig` |
| `configs/oneplus8pro.config` | what is this device's own: the drivers built in, the arguments it needs |
| `soc/` | its SoC family's layer (sm8250): `soc.conf`, `configs/sm8250.config`, the SoC's drivers; carried here so that this one repository is all the device needs |
| `drivers/` | drivers of its own, if it needs any (`drivers/<subsystem>/conf`, as in the common repository) |

Every phone driver of the common repository is left out to begin with: their addresses and pins are the phone they were written
for. Put one back in `configs/oneplus8pro.config` when it has been made to work here, and say so in `device.conf`.

Check with `tools/hakios-device.py check oneplus8pro`.

## sm8250 现状与缺口（2026-11-08）

骨架层面（本次完成）：

- `soc/soc.conf`：vendor=Qualcomm、cores=8（最大核 core 7，fdt 实测：cpu0-3 capacity 1024=A55，
  cpu4-7 capacity 1894=A77，cpu7 为 2.84 GHz prime）、gpu=Adreno 650。
- `soc/configs/sm8250.config`：`CONFIG_SOC/CONFIG_CPUS=8/CONFIG_BOOTCPU=7` 写入；SoC 级驱动开关
  （CPUFREQ/SOCINFO/THERMAL/PMIC）全部关闭并留注释骨架——驱动的内建默认地址是 SC7280 的，
  sm8250 移植写出前打开无意义。
- `soc.args`：16 项补到 **14 项实值**。`soc_cmd_db=0x80860000`（节点名 `reserved-memory@80860000`，
  soc-args.py 按叶子名 `cmd_db` 精确匹配不到，手工补）、`modem_ipa_fw_at=0x8e210000`（kona 把
  ipa_gsi 保留区命名为 `pil_ipa_gsi_region@8e210000`，同理手工补）。
- `tools/hakios-device.py check`：5 个仓库（sc7280/sm8250/oneplus8pro/qemu/redwood）全部通过；
  `list` 正常显示。

缺口（bring-up 顺序）：

- **soc.args 缺 2 项**：`soc_mpss_mem`、`soc_pas_mpss` —— 这棵 OPLUS 树既无 mpss 保留内存、
  也无 `qcom,mss` 节点（PAS id 只有外设的），待实机 bring-up 时补，已注释写明。
- **UEFI**：上游 Mu-Silicium 无 instantnoodlePkg；有 kebabPkg（OnePlus 8T，同为 kona，Inactive 但
  Display/存储/按键/USB Host/Windows Boot 均工作）——以它为模板建平台包；Renegade
  edk2-porting/edk2-msm 上游自带 instantnoodlep 平台与 dtb（2022-11）可作起点/对照。
- **驱动**：21 个手机驱动全部未移植。kona 侧已知差异：触摸按批次为 Synaptics S3908 或 SEC
  S6SY761（树里两节点都在）；Wi-Fi 是 QCA6390（PCIe/MHI，非 redwood 的 WCN6750）；GPU 为
  a650（GMU/固件不同代）；IPA 4.5（kernel 的 IPA 布局是 4.11，蜂窝是最大的移植项）；TSENS v2.4
  地址不同。bring-up 顺序表与每项的 fdt 节点见工作区 `PORT_PLAN.md`。
- **固件**：尚未从手机拉取（pull-firmware.sh 要跑在手机上时执行）；GPU 预期 a650_gmu.bin/a650_zap.mbn，
  WLAN 预期 firmware-qca6390 一组（MHI）。
- **参考树**：devices/redwood 现已 sync（此前 README 生成时缺失），地址对照表可用。

