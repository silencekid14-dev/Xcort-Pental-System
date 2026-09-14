# Pental-System

**A 3D-stacked computing architecture — CPU, GPU, PRAM, and PSSD bonded vertically and linked by the PMPW spatial wire grid.**

[![Status](https://img.shields.io/badge/status-specification-blue)]()
[![Revision](https://img.shields.io/badge/rev-2.0-green)]()
[![Layers](https://img.shields.io/badge/layers-2%20to%2073-orange)]()
[![License](https://img.shields.io/badge/license-see%20LICENSE-lightgrey)]()

---

## What is Pental?

Pental-System is a complete, physically-grounded redesign of the computer stack. Instead of laying silicon flat and pushing clock speeds, Pental **stacks active dies vertically** and wires them with the **Parallel Matrix Physical Wire (PMPW)** grid — a scheduled Manhattan mesh that carries data as physical voltage coordinates rather than serialized packets.

The result is a computing skyscraper with four stacked subsystems:
┌─────────────────────────────────────┐
│ PENTAL-CPU (4–73 layers) │ Compute
├─────────────────────────────────────┤
│ PGPU (4 layers) │ Graphics
├─────────────────────────────────────┤
│ PRAM (8–32 layers) │ Working memory
├─────────────────────────────────────┤
│ PSSD (32–104 layers) │ Persistent storage
├─────────────────────────────────────┤
│ PMPW Bridge (4,096 TSVs, 3.07 TB/s)│ Interconnect
└─────────────────────────────────────┘

text

A hardware **P-Controller** orchestrates the whole stack: it detects idle ALUs, classifies workloads, and migrates programs between layers and between storage tiers — with no OS scheduler required.

---

## The Seven Laws of the Pental System

Every claim in every spec must be derivable from these invariants. If a number contradicts a Law, the number is wrong.

| # | Law | Meaning |
|---|-----|---------|
| **L1** | Law of Verticality | Computation is a skyscraper, not a plane. Every block must justify not being stacked. |
| **L2** | Law of the Matrix Wire | Data is a voltage coordinate on a physical wire, not a serialized packet in a queue. |
| **L3** | Law of Isolation | Each layer is an independent compute island. No crash, thermal event, or clock stall propagates. |
| **L4** | Law of Thermal Escape | Heat travels *sideways* before it travels up. Vertical heat flow through silicon is a bottleneck. |
| **L5** | Law of Adaptive Coordination | Uniform work → STET. Divergent work → DTET. Hardware switches without software intervention. |
| **L6** | Law of Spatial Memory | Memory is a coordinate grid, not a request-response queue. Address = physical coordinate. |
| **L7** | Law of the P-Controller | No ALU idles if work exists anywhere in the stack. Idle detection is hardware's job. |
| **L8** | Storage Corollary | Hot data rises; cold data sinks. Data migrates vertically by access frequency, like heat. |

---

## Repository Contents

| File | Subsystem | What's inside |
|------|-----------|---------------|
| [`PENTAL-CPU-1.md`](./PENTAL-CPU-1.md) | **Compute** | ISA (PENTAL-X), 8-stage pipeline, cache hierarchy, directory MESI, PMPW arbitration (STDM + token ring), Throlter thermal model, power delivery, boot/debug/security |
| [`PGPU.md`](./PGPU.md) | **Graphics** | Hybrid SIMD/MIMD engine, hash-bucket Shader Scheduler, fixed-function + general ALU block diagram, STET/DTET switch cost, realistic utilization |
| [`PRAM.md`](./PRAM.md) | **Working Memory** | Spatial coordinate memory, mandatory layer-select fix, write RC model, sense-amp budget, refresh accounting, ECC + scrubbing |
| [`PSSD.md`](./PSSD.md) | **Storage** | Tiered MRAM → ReRAM → NAND stack, hardware FTL, sub-block allocation, hot/cold migration |
| [`P-controller.md`](./P-controller.md) | **Orchestration** | Hardware idle detection, program classification, layer assignment, migration engine, OS interface |

---

## Headline Numbers

### Compute (4-Layer Desktop)
| Metric | Value |
|---|---|
| Layers / Cores / ALUs | 4 / 16 / 64 |
| Clock | 5.0–5.2 GHz (per-layer DVFS) |
| FP32 throughput | ~52 TFLOP/s |
| Power | 80–160 W |
| L3 (PMPW SRAM) | 32 MB shared |

### Graphics (PGPU)
| Metric | Value |
|---|---|
| ALUs | 16,384 (128-bit SIMD lanes) |
| Clock | 6.0 GHz |
| FP32 throughput | ~786 TFLOP/s |
| Path-traced 4K frame | ~2 ms |
| ALU utilization (divergent) | 72% (vs ~25% on standard GPUs) |
| Power | ~340 W |

### Working Memory (PRAM)
| Metric | Value |
|---|---|
| Capacity | 16 GB base, 64 GB max |
| Read latency | ~4.0 ns |
| Streaming bandwidth | 3.07 TB/s |
| Random transaction rate | ~384 GB/s |
| Refresh overhead | 8% (honest, not 0%) |

### Storage (PSSD)
| Metric | Value |
|---|---|
| Capacity | 4.6 TB (desktop), 32 TB (supercomputer) |
| T0 (MRAM) latency | **11 ns read / 11 ns write** |
| T1 (ReRAM) latency | 101 ns / 1 µs |
| T2 (NAND) latency | 50 µs / 500 µs |
| Random 4K IOPS | 750 M (T0) |
| Power | 120 W active / 1 W idle |

---

## Why This Exists

Modern computing has three walls:

1. **The Von Neumann bottleneck** — CPUs wait on packetized memory buses.
2. **The Thermal Wall** — 2D silicon cannot dissipate more heat without exotic cooling.
3. **The Divergence Problem** — GPUs force divergent threads into lockstep SIMD, idling 60–80% of ALUs.

Pental attacks all three with **physical topology**, not software tricks:

- **Vertical stacking** replaces the 2D floorplan with a 3D skyscraper.
- **The PMPW grid** replaces queues with scheduled physical wires.
- **Layer isolation** replaces warp divergence with independent compute islands.
- **The P-Controller** replaces OS scheduling with hardware orchestration.
- **Tiered non-volatile storage** replaces the NVMe packet queue with spatial coordinates.

---

## Design Discipline

Every spec in this repo obeys a single rule: **physics-level claims must be derivable from real material constants.** No hand-waving. Where the numbers don't close, the spec says so.

Examples of honest corrections applied in Revision 2.0:

- The original 73-layer stack height (7.3 mm) was replaced with a **thinned-die** model (1.33 mm) that achieves a 13:1 TSV aspect ratio instead of an impossible 365:1.
- The "0% refresh overhead" claim for PRAM was corrected to **8.1%**, with the bandwidth arithmetic shown.
- The PGPU triangle-setup rate was corrected from **98 B/s to 1.3 B/s** (fully shaded), with the op-count per triangle shown.
- The Throlter sheet's thermal model was corrected: a continuous 20 mm graphene sheet would require **3,300 K** of lateral ΔT; the fix is **patterned graphene pillars** at 1,024/mm².
- The PRAM layer-select hole (a TSV passing through all 8 layers would turn on 8 cells at once) was closed with a mandatory **Layer-Select line** at a **17% die-area cost**.

---

## Status

| Milestone | State |
|---|---|
| Architecture specification | ✅ Revision 2.0 complete |
| Thermal model with real constants | ✅ Complete |
| ISA + microarchitecture | ✅ Defined (PENTAL-X + x86-64 legacy) |
| Coherence protocol | ✅ Directory MESI |
| PMPW arbitration | ✅ STDM + token ring |
| Hardware FTL | ✅ Complete |
| P-Controller orchestration | ✅ Complete |
| RTL implementation | ⬜ Not started |
| Silicon prototype | ⬜ Not started |
| Simulation results | ⬜ Not started |

This is a **specification repository**, not an implementation. It is intended as an engineering blueprint, a teaching artifact, and a starting point for anyone who wants to build a 3D-stacked machine.

---

## Roadmap

- **Phase 1 (this repo):** Specification, physics models, arithmetic closure.
- **Phase 2:** Cycle-accurate simulator (gem5 fork or custom) for the PMPW grid.
- **Phase 3:** RTL for a single-layer prototype on an FPGA.
- **Phase 4:** TSMC CyberShuttle test chip for one PMPW layer.
- **Phase 5:** Multi-layer SoIC bonding with TSMC or imec.

---

## Contributing

Contributions that **close holes** are the highest priority. If you find a claim that contradicts one of the Seven Laws — or a number that doesn't close — open an issue with:

1. The file and section.
2. The claim as written.
3. The physics-level contradiction.
4. A proposed correction or a request for clarification.

Pull requests that add **honest numbers** (real material constants, real cell latencies, real TSV budgets) are welcome. Pull requests that add marketing language are not.

---

## How to Read the Specs

Each file is written to be self-contained but references the others:
PENTAL-CPU-1.md ← start here (defines PMPW, Laws, ISA)
├── PGPU.md (inherits PMPW + STET/DTET)
├── PRAM.md (inherits PMPW, defines L6 storage)
├── PSSD.md (inherits PRAM topology, adds tiers)
└── P-controller.md (orchestrates all four)

text

If you only read one file, read `PENTAL-CPU-1.md` §0 (Laws) and §3 (PMPW arbitration). Everything else is downstream of those two sections.

---

## FAQ

**Q: Is this real hardware?**
A: No. This is a specification. No silicon has been fabricated. The numbers are grounded in real material constants and real process-node capabilities (2nm BEOL, TSMC SoIC, HBM-class die thinning), but no foundry has built a Pental chip.

**Q: Why 4 layers for the desktop and 73 for the supercomputer?**
A: 4 layers fits a consumer package with air or AIO cooling at 80–160 W. 73 layers requires direct-to-silicon microfluidics and 1,460–2,920 W, and is targeted at planetary-scale AI data centers.

**Q: Is PRAM just DRAM?**
A: No. PRAM uses the same 1T1C cell but replaces the packetized DDR bus with a spatial coordinate grid. The cell is standard; the *interconnect* is the invention.

**Q: Why not use HBM instead?**
A: HBM is a 2.5D interposer — the dies are side-by-side. Pental is 3D — the dies are stacked. HBM's interposer is a packet bus; Pental's PMPW is a coordinate grid.

**Q: Does the P-Controller replace the OS scheduler?**
A: No. It runs *underneath* the OS scheduler. It exposes memory-mapped registers the OS can read, but it acts autonomously to route work to idle layers even if the OS ignores it.

**Q: What about security? The PMPW is a shared bus.**
A: The STDM schedule is fixed and data-independent, so wire activity does not leak data values. Standard Spectre/Meltdown mitigations apply in microcode. See `PENTAL-CPU-1.md` §6.5.

---

## License

This repository contains the engineering specifications and architectural blueprints for the Pental-System.

* The entire repository is licensed under the **CERN Open Hardware Licence Version 2 – Weakly Reciprocal** ([CERN-OHL-W-2.0](https://spdx.org/licenses/CERN-OHL-W-2.0.html)).

By using, modifying, or distributing these specifications, you agree to the reciprocal terms of this license. Any derivative hardware layouts or architecture extensions must be made public under these identical terms.


---

## Acknowledgments

This specification was developed under the discipline of the **Seven Laws of the Pental System**. The audit-and-correction methodology (identify the hole, state the physics contradiction, propose the fix, recompute the number) is the core intellectual contribution of Revision 2.0.

*The Pental System — compute is a skyscraper.*