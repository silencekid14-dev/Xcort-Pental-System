The PENTAL CPU Architecture Specification
Complete Microscopic 3D-Stacked Parallel Matrix Processor Blueprint
Revision 2.0 — Extended Engineering Edition
0. The Laws of the Pental System
Every claim in every Pental document must be derivable from these seven invariants. If a number contradicts a Law, the number is wrong.

#	Law	Meaning
L1	Law of Verticality	Computation is a skyscraper, not a plane. Every functional block must justify why it is not stacked.
L2	Law of the Matrix Wire	Data is a voltage coordinate on a physical wire, not a serialized packet in a queue.
L3	Law of Isolation	Each layer is a physically independent compute island. A crash, thermal event, or clock stall on one layer must not propagate to another.
L4	Law of Thermal Escape	Heat must travel sideways (lateral) before it travels up (vertical). Vertical heat flow through silicon is a bottleneck, not a highway.
L5	Law of Adaptive Coordination	When work is uniform, use STET (Same Task Exact Time). When work diverges, use DTET (Different Tasks Exact Time). The hardware must switch without software intervention.
L6	Law of Spatial Memory	Memory is a coordinate grid, not a request-response queue. Address = physical coordinate.
L7	Law of the P-Controller	No ALU idles if work exists anywhere in the stack. Idle detection and program assignment are hardware responsibilities, not OS responsibilities.
1. Executive Summary & Design Philosophy
The PENTAL CPU is a 3D-stacked computer architecture that replaces the Von Neumann bottleneck with a physical computing skyscraper. Multiple active computational layers are stacked vertically and wrapped in a high-density routing fabric — the Parallel Matrix Physical Wire (PMPW) grid — that carries data as continuous voltage coordinates rather than serialized packets.

PENTAL is optimized for two simultaneous realities:

General operating workloads (Windows 11, GTA, Minecraft) that demand per-layer hardware isolation.

AI vector math (matrix multiply, attention, convolution) that demands massive uniform throughput.

Revision 2.0 adds the missing engineering substrate: the instruction set, the pipeline, the cache hierarchy, the coherence protocol, the arbitration scheme for PMPW, the thermal resistance model for the Throlter sheet, the power delivery network, and the boot/debug/security fabric. It also corrects the 73-layer arithmetic to physically realizable numbers.

2. Hardware Architecture & Layer Stacking
2.1 The 4-Layer Desktop Specification
Parameter	Value
Active silicon layers	4
Cores per layer	4
ALUs per core	4 (128-bit SIMD datapath each)
Total cores	16
Total ALUs	64
Total raw FLOPs (FP32)	64 ALU × 2 (FMA) × 128 bit/32 bit × 5.1 GHz ≈ 52 TFLOP/s
Manufacturing node	2nm–5nm
Clock	5.0–5.2 GHz (per-layer DVFS)
Power	80–160 W gaming load
L1 per core	64 KB I + 64 KB D
L2 per layer	2 MB (shared by 4 cores)
L3 (PMPW SRAM)	32 MB shared across all 4 layers
PRAM (stacked below)	16 GB base, 64 GB max
2.2 The 73-Layer Supercomputing Specification — Corrected
The original spec claimed 73 layers × 100 µm = 7.3 mm. That is physically impossible with current TSV aspect ratios. Revision 2.0 uses thinned-die stacking:

Parameter	Original claim	Corrected reality
Layer thickness	100 µm	10 µm (post-thinning, hybrid-bonded)
Stack height	7.3 mm	0.73 mm active + 0.6 mm Throlter = 1.33 mm
TSV aspect ratio	~365:1 (impossible)	~13:1 (achievable today)
Layers	73	73 (unchanged)
Cores	292	292
ALUs	1,168	1,168
Clock	2.5–3.5 GHz	2.5–3.5 GHz (unchanged — thermal)
Power	1,460–2,920 W	1,460–2,920 W (unchanged)
Cooling	"microfluidic"	Direct-to-silicon microfluidic + graphene lateral escape
The 73-layer number is kept, but the stack is thin because each die is thinned to 10 µm before bonding — the same technique TSMC uses for HBM.

2.3 Layer-by-Layer Block Diagram
text
      ┌─────────────────────────────────────────────────────────┐
      │   LAYER 4: OS & Thread Watchdog  (Ring 0, UEFI, APIC)   │
      ├─────────────────────────────────────────────────────────┤  ◄── Throlter Sheet 3
      │   LAYER 3: Complex Structured Math (Minecraft chunk math)│
      ├─────────────────────────────────────────────────────────┤  ◄── Throlter Sheet 2
      │   LAYER 2: Linear Logic & Scripting (GTA physics/scripts)│
      ├─────────────────────────────────────────────────────────┤  ◄── Throlter Sheet 1
      │   LAYER 1: PMPW Interconnect & GPU Dispatch             |  
      └──────────────────────────┬───┬──────────────────────────┘
                                 │   │
                                 ▼   ▼
                     Through-Silicon Vias (TSVs)
                     [To PRAM stack below + motherboard pins]
3. The Parallel Matrix Physical Wire (PMPW) Grid — With Arbitration
3.1 Topology
The PMPW is a two-dimensional Manhattan mesh per layer, plus a vertical TSV elevator bank that connects all layers.

Horizontal "Street" bus (X-axis): 512 wires per layer, running left→right.

Horizontal "Avenue" bus (Y-axis): 512 wires per layer, running front→back.

Vertical TSV elevators: 4,096 columns, running bottom→top through every layer.

Per-ALU tap: Each ALU has a 128-bit tap into one X wire and one Y wire via a crosspoint switch.

3.2 The Arbitration Problem (Previously Unaddressed)
The audit's #1 hole: the original spec claimed "no packetized arbitration — continuous hardwired voltage paths." That is only true if no two drivers ever contend for the same wire. PENTAL Revision 2.0 makes this true by construction:

3.2.1 Static Time-Division Multiplexing (STDM)
Each of the 512 X wires and 512 Y wires on a layer is assigned a fixed round-robin slot schedule at fabrication time. The schedule is stored in a 64-entry ROM per wire. At cycle T mod 64 = k, only the ALU whose ID equals k may drive that wire. All other ALUs tri-state.

Cost: a 3-bit slot counter + a 64:1 grant lookup per wire. ~200 gates per wire × 1,024 wires = ~200k gates per layer. Trivial.

Benefit: zero arbitration latency. The wire is never contested because the schedule is fixed.

Consequence: an ALU may only transmit once every 64 cycles on its X wire. To sustain 128-bit/cycle throughput, each ALU is given 4 parallel X wires (staggered slots) and 4 parallel Y wires. That is 8 wires × 64 ALUs = 512 wires per layer — which matches the 512-wire budget above.

3.2.2 Vertical TSV Arbitration (Token Ring)
Vertical TSVs are shared across all 4 layers. Contention is resolved by a distributed token ring:

A single "token bit" circulates through the 4 layers on a dedicated 1-bit vertical wire.

The layer holding the token may drive all 4,096 TSVs for 1 cycle.

The token passes to the next layer on the following cycle.

Result: each layer gets 1 out of every 4 cycles of vertical bandwidth. Effective vertical bandwidth per layer = 3.07 TB/s ÷ 4 = 768 GB/s per layer, 3.07 TB/s aggregate.

This is the missing mechanism that makes "no queue" true: it is a deterministic schedule, not a queue.

3.3 Wire Count Arithmetic (Previously Hand-Waved)
The audit correctly noted that 64 ALUs × 512-bit operands = 32,768 wires is absurd. Revision 2.0 closes this:

Resource	Count	Justification
ALU datapath width	128 bits (not 512)	128-bit FMA is the 2nm sweet spot
X wires per layer	512	64 ALUs × 4 staggered slots × 2 (in/out)
Y wires per layer	512	Same
Vertical TSVs	4,096	1 bit per TSV per cycle
Wire pitch (X/Y)	40 nm	2nm BEOL, metal layer 8
Bus width	512 × 40 nm = 20.5 µm per bus	Fits easily on a 20 mm die
Repeater spacing	Every 500 µm	5 GHz needs insertion every ~500 µm at 2nm
Conclusion: the PMPW is a scheduled Manhattan mesh, not a "continuous voltage soup." It is faster than a packet bus because the schedule is fixed and there is no queue, but it is not magic.

4. Core Microarchitecture (Previously Missing)
4.1 The PENTAL-X Instruction Set
PENTAL cores natively execute x86-64 (for Windows 11 / GTA / Minecraft compatibility) plus a PENTAL-X extension for PMPW matrix ops.

Legacy path: x86-64 decoded by a 4-wide decoder. One PENTAL core = one x86-64 core. The "4 ALUs" are the 4 SIMD lanes of a 128-bit AVX-512-compatible unit (Intel's AVX-512 uses 512-bit = 4 × 128-bit lanes).

PENTAL-X path: new opcodes PMADD, PMROUTE, PMSYNC, PMLAYER. These operate on the PMPW grid directly.

PMADD dst, srcA, srcB, layer_mask — FMA on all ALUs whose layer matches layer_mask.

PMROUTE src, dst_layer, dst_zone — push a 128-bit value to a specific layer/zone via TSV.

PMSYNC barrier_id — cross-layer barrier.

PMLAYER rd, layer_id — read the current layer's ID.

Binary translation: legacy x86-64 binaries run natively. PENTAL-X code requires recompilation. The LLVM backend llvm-pental is defined in §9.

4.2 Pipeline
Stage	Function	Cycles
F	Instruction fetch (I-cache, 64 KB)	1
D	Decode (4-wide)	1
R	Rename (256 physical registers)	1
S	Schedule (out-of-order, 128-entry ROB)	1
I	Issue	1
E	Execute (4 ALU lanes)	1
M	Memory (L1D 64 KB, 4-cycle latency)	1
W	Writeback	1
Pipeline depth: 8 stages. At 5.1 GHz, 8 stages is shallow; the 2nm node allows it. Branch misprediction penalty: 8 cycles.

4.3 Branch Prediction
TAGE-SC-L predictor, 64 KB, 3-level.

BTB: 4,096 entries.

Return stack: 32 entries.

Accuracy target: ≥ 97% on SPECint, ≥ 99% on game loops.

4.4 Out-of-Order Execution
ROB: 128 entries.

Issue width: 4.

Load/store queue: 64 entries.

Physical registers: 256 (int) + 256 (vector).

4.5 Register File
32 × 64-bit architectural GPRs (x86-64).

32 × 512-bit vector registers (AVX-512).

256 × 64-bit physical GPRs.

256 × 512-bit physical vector registers.

5. Cache Hierarchy & Coherence (Previously Missing)
5.1 The Hierarchy
Level	Size	Latency	Scope
L1I	64 KB	4 cycles	Per core
L1D	64 KB	4 cycles	Per core
L2	2 MB	12 cycles	Per layer (shared by 4 cores)
L3 (PMPW SRAM)	32 MB	24 cycles	Shared across all 4 layers
PRAM	16–64 GB	~4 ns (~24 cycles at 6 GHz)	Stacked below
The L3 lives inside the PMPW grid, distributed as 8 MB per layer, with a cross-layer directory. This is the "PMPW SRAM" PRAM.md referred to but never sized.

5.2 Cache Line Size
64 bytes (x86 standard). This determines the vertical TSV transaction size: a cache line fill = 64 B = 512 bits = 512 TSVs × 1 cycle.

5.3 Coherence Protocol — Directory-Based MESI
The audit's #2 hole: cross-layer coherence was never specified.

Protocol: directory-based MESI, with the directory stored in the L3 (PMPW SRAM).

States: Modified, Exclusive, Shared, Invalid.

Directory: one entry per 64 B line, 512 M entries for 32 MB L3 → 4 MB directory. Stored in a dedicated SRAM region of Layer 1.

Snoop filter: the directory acts as a snoop filter; no broadcast.

Cross-layer invalidation: when Layer 1 writes a line cached in Layer 3, the directory sends an INVAL packet over the PMPW Y wires to Layer 3's L2. Latency: ~8 cycles.

Why not snooping? With 4 layers × 4 cores = 16 cores, a broadcast snoop would saturate the PMPW. Directory is mandatory at this scale.

5.4 The Memory Model
Sequential consistency for PENTAL-X ops, TSO (Total Store Order) for x86-64 legacy ops — matching x86 semantics so Windows 11 runs unmodified.

6. Interrupts, MMU, Boot, Debug, Security (Previously Missing)
6.1 Interrupts
APIC: one per layer, located in Layer 4 (OS layer).

Cross-layer IPI: via PMROUTE to a dedicated 4-bit interrupt TSV.

Latency: ~16 cycles layer-to-layer.

6.2 MMU / TLB
Per-core TLB: 64-entry L1, 2,048-entry L2.

Shared page tables in PRAM.

Page sizes: 4 KB, 2 MB, 1 GB.

Virtualization: nested paging (EPT) for Hyper-V.

6.3 Boot & Firmware
UEFI: runs on Layer 4 (the OS layer). Layer 4 has a dedicated 16 MB ROM.

Secure boot: Layer 4 verifies Layer 3, Layer 2, Layer 1 firmware hashes before releasing reset.

Reset tree: global reset via a dedicated TSV; per-layer reset via PMROUTE.

6.4 Debug & Test
JTAG: 5-pin, accessible via package edge.

On-die trace: 4,096-entry trace buffer per layer, readable via JTAG.

Performance counters: 8 per core, 32 per layer.

Known-good-die (KGD) test: each layer is probed at wafer level before bonding. A dead layer is fused off; the chip boots with 3/4 layers.

6.5 Security
PMPW side-channel: the PMPW is a shared bus, so it is a side-channel risk. Mitigation: constant-time PMPW scheduling — the STDM schedule (§3.2.1) is fixed and data-independent, so wire activity does not leak data values.

Spectre/Meltdown: standard mitigations (retpoline, IBRS) in microcode.

Encryption: AES-NI in every ALU lane.

7. Advanced Interlayer Thermal Management: The "Throlter" Sheet (With Real Numbers)
7.1 Structural Configuration
text
 ┌─────────────────────────────────────────────────────────────┐
 │               Wafer Layer (ALUs + PMPW Grid)                │
 ├─────────────────────────────────────────────────────────────┤ ◄── Microscopic Isolation Teeth
 │  ─── Layer A: h-BN (Electrical Insulator, 200 nm) ───────── │
 │  ─── Layer B: Graphene Nano-Ribbons (Thermal Highway, 5 µm) │
 │  ─── Layer C: h-BN (Electrical Insulator, 200 nm) ───────── │
 ├─────────────────────────────────────────────────────────────┤
 │                  Subsequent Wafer Layer                     │
 └─────────────────────────────────────────────────────────────┘
7.2 The Thermal Resistance Model (Previously Unproven)
The audit correctly noted that h-BN cross-plane conductivity is ~2–4 W/m·K — a bottleneck, not a highway.

Layer	Thickness	k (W/m·K)	R'' (m²K/W)	ΔT for 10 W/mm²
Si (active)	10 µm	150	6.7×10⁻⁸	0.7 K
h-BN (top)	200 nm	3	6.7×10⁻⁸	0.7 K
Graphene (lateral)	5 µm	3000 (in-plane)	1.7×10⁻⁹	0.02 K
h-BN (bottom)	200 nm	3	6.7×10⁻⁸	0.7 K
Cu spreader	500 µm	400	1.25×10⁻⁶	12.5 K
Key insight: the h-BN layers are thin enough (200 nm) that their thermal resistance is comparable to the silicon itself. The real bottleneck is the lateral distance to the die edge. To move 100 W from the center of a 20 mm die to the edge:

Graphene cross-section: 5 µm × 20 mm = 1×10⁻⁷ m².

Thermal resistance: 10 mm / (3000 × 1×10⁻⁷) = 33 K/W.

For 100 W: 3,300 K — impossible.

Fix: the Throlter sheet is not a single 20 mm graphene sheet. It is patterned into 1,024 vertical graphene pillars per mm², each 5 µm tall, connecting the silicon directly to a copper spreader directly above each core. This reduces the lateral distance from 10 mm to ~200 µm (the pillar pitch), dropping the thermal resistance by 50×.

Corrected path	ΔT for 10 W/mm²
Si → h-BN → graphene pillar → Cu (200 µm lateral)	~5 K
vs. original 10 mm lateral graphene	~3,300 K
Verdict: the Throlter sheet works only if it is patterned into short pillars, not a continuous sheet. Revision 2.0 makes this explicit.

7.3 DTI Teeth — Corrected Aspect Ratio
The audit noted that reaching active silicon through a 10 µm thinned die requires a 250:1 aspect ratio. Revision 2.0 uses post-thinning DTI: the DTI is etched after the die is thinned to 10 µm, giving an aspect ratio of ~50:1 (10 µm deep, 200 nm wide). This is achievable with modern Bosch-process etch.

8. Power Delivery (Previously Missing)
Input: 12 V from motherboard via 8-pin EPS connector.

Per-layer VRM: each layer has an integrated voltage regulator (IVR) on its edge, stepping 12 V → 0.75 V.

Vertical power TSVs: 256 dedicated power TSVs, 2 A each = 512 A capacity.

DVFS: per-layer. Layer 2 can run at 5.2 GHz while Layer 3 runs at 4.8 GHz.

IR drop control: 32 on-die voltage sensors per layer feed a digital LDO that compensates within 2 cycles.

Power gating: each ALU can be individually gated via PMLAYER mask.

9. Memory Architecture & Vertical Vias
9.1 Through-Silicon Vias (TSVs)
4,096 data TSVs at 15 µm pitch.

Distributed evenly across the die (not edge-confined).

Bonded to PRAM below via TSMC SoIC hybrid bonding.

9.2 Multi-Channel RAM Isolation
Channel D: Layer 4 (OS, background).

Channel C: Layer 3 (Minecraft chunk math).

Channel B: Layer 2 (GTA physics/scripts).

Channel A: Layer 1 (GPU dispatch).

Each channel is 1,024 TSVs → 768 GB/s per channel.

10. Software & Real-World Operating Workloads
10.1 Windows 11 + GTA + Minecraft
Layer 4: Windows 11 kernel, Discord, browsers. Isolated from gaming layers.

Layer 3: Minecraft chunk generation, matrix-mapped to PMPW vectors.

Layer 2: GTA physics and script triggers, single-threaded boost to 5.2 GHz.

Layer 1: GPU dispatch and frame packing.

10.2 The P-Controller
See P-controller.md. The P-Controller monitors ALU utilization per layer and migrates programs to idle capacity. This is hardware, not OS scheduling — it satisfies L7.

11. Manufacturing & Foundry Feasibility
11.1 4-Layer Model
TSMC SoIC + 3D Fabric, available today. TSMC CyberShuttle for prototyping.

11.2 73-Layer Model
Requires:

10 µm die thinning (HBM-class, available).

13:1 TSV aspect ratio (available).

Sub-1 µm hybrid bonding alignment across 20 mm (TSMC SoIC is at ~0.5 µm — available).

Microfluidic cooling channels between every 8th layer (research-stage).

Realistic timeline: 2035–2040, not 2050.

***End of PENTAL-CPU-1.md — Revision 2.0***