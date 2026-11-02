# UniXNU-sm8475-common

Shared platform support for **Qualcomm SM8475** (Snapdragon 8+ Gen 1, "cape" / "taro").

Boards include: Redmi K50 Ultra / Xiaomi 12T Pro (`diting`), Xiaomi Pad 6 Pro (`liuqin`),
OnePlus Ace 2 / 11R (`PHK110`).

## Scope

| What | Where |
|---|---|
| SoC identity and the components this SoC selects | `soc.toml` |
| Shared kernel config fragment | `configs/sm8475.config` |
| Shared hardware description (CPUs, GIC, UFS, GPU/GMU/SMMU) | `dts/sm8475.dtsi` |
| SoC XNU patches, in order | `patches/xnu/series` (empty for now) |
| Shared platform operations | `kits/` (empty for now) |

A board's DTS includes `sm8475.dtsi` and adds its own memory, reserved regions and peripheral
wiring.

## Where the numbers come from

Two sources, cross-checked against each other:

1. **The phone's own `vendor_boot` device tree**, `qcom,cape-v2`, extracted from the Xiaomi fastboot
   ROM `OS1.0.16.0.ULFCNXM`.
2. **The running phone over adb** — `/sys/bus/platform/devices/`, whose node names carry the base
   addresses (`ed18000.qcom,ipcc`, `15000000.apps-smmu`, `1d84000.ufshc`, …).

A third source is recorded for comparison: `boomoodxf/UniXNU-ace2`, a bring-up of the OnePlus Ace 2
on this same SoC that reached XNU entry on hardware. Its `board.toml` values agree with ours:
GIC `0x17100000` / `0x17180000` / stride `0x40000`, UFS `0x1d84000` / `0x1d80000`, framebuffer
`0xb8000000`.

## What is deliberately not here yet

- **No GPU component.** `components.include` lists only `ArmCpu` and `ArmDTPlatform`. The tree's
  native Adreno provider is `Adreno740` (SM8550); SM8475 has an Adreno 730 and no provider for it
  has been written. The GPU/GMU/SMMU nodes in `sm8475.dtsi` are therefore `status = "disabled"`.
- **No SoC XNU patches.** `patches/xnu/series` is intentionally empty rather than copied from
  SM8550 — its two patches (`native-smp-secondary`, `sheng-platform-power`) are written against
  SM8550 behaviour and have not been shown to apply here.

## Known differences from SM8550 that matter

| Item | SM8550 | SM8475 |
|---|---|---|
| CPU topology | 1×X3 + 2×A715 + 2×A710 + 3×A510 | **1×X2 + 3×A710 + 4×A510** |
| GPU | Adreno 740 | **Adreno 730** |
| GMU third window | `0xb280000` | **`0xb290000`** |
| PMIC family | SM8550's | **pm8350/pm8350b/pm8350c/pm8450/pmk8350/pmr735a** |

The PMIC point is why Mu-Silicium's `WaipioPkg` (SM8450's SoC package) is the right base for this
SoC, not a package from the SM8550 generation.
