# AGENTS.md — UniXNU-sm8475-common

Shared platform support for Qualcomm SM8475. Read this before changing anything here.

## What belongs here

Only what is true of **every** SM8475 board:

- CPU topology and PSCI
- GIC (distributor, redistributor base and stride)
- UFS controller and PHY base addresses
- GPU / GMU / GPU-SMMU register windows
- Shared kernel config fragment and SoC XNU patches

Memory maps, reserved regions, panels, touch controllers, PMIC rails and partition geometry belong
to the **board** repository (`UniXNU-<device>`), not here.

## Rules

1. **Every address needs a source.** Acceptable sources, in order: the board's own
   `vendor_boot`/`dtbo` device tree read from the device or its vendor fastboot ROM; the running
   device over `adb`; a cross-check against a second board on the same SoC. Never copy a value from
   another SoC's `dtsi` — SM8550 differs in the CPU topology, the GPU model and the GMU's third
   window.
2. **Never enable a node you cannot justify.** The GPU/GMU/SMMU nodes ship `status = "disabled"`
   because no Adreno 730 provider exists in the tree yet. Flipping them on without a provider
   produces a board that builds and does not boot.
3. **`patches/xnu/series` stays honest.** An empty series is a valid, deliberate statement. Do not
   fill it with SM8550's patches to look complete.
4. **A component in `soc.toml` must exist in the component catalogue.** Listing `Adreno730` before
   anyone writes it makes every board that selects this SoC fail to resolve.
5. Record the evidence for a change in the commit message: the command that produced it, or the
   node path in the device tree it came from.

## References

- `docs.toml`/`soc.toml` schema: project docs, "选择与添加 SoC 支持"
- Board side: `UniXNU-<device>/AGENTS.md`
- Example implementation: `UniXNU-sm8550-common`
