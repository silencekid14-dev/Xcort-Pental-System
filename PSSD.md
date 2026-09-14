File 5 — PSSD.md
PSSD.md — Pental Solid State Drive
The 3D-Stacked Non-Volatile Spatial Storage Architecture
Revision 1.0 — Engineering Edition
0. The Laws Applied to Storage
The PSSD inherits all Seven Laws of the Pental System and adds one storage-specific corollary:

Law	Storage interpretation
L1 (Verticality)	Storage is a skyscraper of non-volatile tiers, not a planar NAND chip.
L2 (Matrix Wire)	A logical block address (LBA) is a physical voltage coordinate, not a packet in an NVMe queue.
L3 (Isolation)	Each storage tier is a physically independent zone. A NAND erase stall must not block MRAM reads.
L4 (Thermal Escape)	NAND program/erase generates heat; the Throlter pillar network pulls it sideways.
L5 (Adaptive Coordination)	Bulk sequential writes use STET; random 4K writes use DTET.
L6 (Spatial Memory)	The FTL is a hardware coordinate translator, not a software lookup table.
L7 (P-Controller)	No storage tier idles if there is pending I/O anywhere in the stack.
L8 (Storage Corollary)	Hot data rises; cold data sinks. Data migrates vertically between tiers based on access frequency, exactly as heat migrates laterally in the Throlter sheet.
1. Executive Summary & The NVMe Crisis
Modern NVMe SSDs are planar. A 4 TB drive is 4–8 NAND dies wire-bonded to a controller, all sitting flat on a PCB. The controller runs a software FTL (Flash Translation Layer) on an embedded ARM core, translating 4K logical blocks to physical NAND pages through a DRAM lookup table. Every I/O goes through:

PCIe transaction layer (packetized)

NVMe command queue (submission/completion)

Controller firmware (FTL lookup, wear leveling, GC)

NAND ONFI bus (8–16 bits wide, 1–2 GHz)

Page buffer → NAND array → sense amp

Latency: 20–100 µs for a 4K read. That is 5,000–25,000× slower than PRAM. The bottleneck is not the NAND cell — it is the packetized queue and software FTL.

The PSSD eliminates all of it. By extending the PMPW grid into the storage stack:

LBA = physical coordinate (no FTL lookup table).

4,096 parallel TSVs (no 8-bit ONFI bus).

Hardware FTL (no firmware ARM core).

Tiered non-volatile cells (MRAM → ReRAM → NAND) with automatic hot/cold migration (Law L8).

Result: 4K read latency drops from ~80 µs to ~200 ns (MRAM tier) or ~2 µs (ReRAM tier). NAND tier remains ~50 µs but is only used for cold data.

2. The Tiered Storage Hierarchy
The PSSD is not one technology. It is a vertical stack of three non-volatile tiers, each optimized for a different access pattern. This mirrors the CPU's SRAM/L3/PRAM hierarchy, but for persistence.

Tier	Technology	Layers	Capacity/layer	Read latency	Write latency	Endurance	Role
T0 — Fast	MRAM (STT-MRAM)	4	16 GB	10 ns	10 ns	10¹⁵ cycles	OS boot, metadata, journal
T1 — Medium	ReRAM (OxRAM)	16	64 GB	100 ns	1 µs	10⁹ cycles	Active user data, games
T2 — Cold	3D NAND (QLC)	12	256 GB	50 µs	500 µs	10³ cycles	Archives, backups, video
Total	—	32	—	—	—	—	~4.6 TB
The P-Controller (see P-controller.md) migrates data vertically between tiers based on access frequency. Hot data rises to T0; cold data sinks to T2. This is Law L8.

3. Physical Architecture
3.1 Layer Stacking
text
┌─────────────────────────────────────────────────────────────┐
│  PSSD LAYER 31 (T2 — NAND, 256 GB)                          │
│  ...                                                         │
│  PSSD LAYER 20 (T2 — NAND, 256 GB)                          │
├─────────────────────────────────────────────────────────────┤ ◄── Throlter Sheet
│  PSSD LAYER 19 (T1 — ReRAM, 64 GB)                          │
│  ...                                                         │
│  PSSD LAYER 4  (T1 — ReRAM, 64 GB)                          │
├─────────────────────────────────────────────────────────────┤ ◄── Throlter Sheet
│  PSSD LAYER 3  (T0 — MRAM, 16 GB)                           │
│  ...                                                         │
│  PSSD LAYER 0  (T0 — MRAM, 16 GB)                           │
├─────────────────────────────────────────────────────────────┤
│  PMPW BRIDGE (4,096 TSVs to CPU/PRAM stack above)           │
└─────────────────────────────────────────────────────────────┘
3.2 Physical Dimensions
Parameter	Value
Layer thickness	10 µm (thinned, hybrid-bonded)
Throlter sheets	Every 4 layers (8 sheets total)
Total stack height	320 µm active + 80 µm Throlter = 400 µm
Die size	20 mm × 20 mm
TSV count	4,096 data + 256 control
TSV pitch	15 µm
Package	24 mm × 24 mm × 0.6 mm
3.3 The PMPW Bridge
The PSSD connects to the CPU/PRAM stack via a PMPW bridge — a silicon interposer with 4,096 copper traces, 20 mm long, running at 6 GHz.

Bridge bandwidth: 4,096 bits × 6 GHz = 3.07 TB/s (same as PRAM).

Bridge latency: ~200 ps (20 mm at ~0.1 c).

Protocol: identical STDM schedule as the CPU PMPW grid (see PENTAL-CPU-1.md §3.2.1).

4. Cell Technology Details
4.1 T0 — MRAM (STT-MRAM)
Structure: magnetic tunnel junction (MTJ) — two ferromagnetic layers separated by a 1 nm MgO barrier. Resistance is high (anti-parallel) or low (parallel).

Parameter	Value
Cell size	6F² (at 2nm: ~20 nm²)
Read current	10 µA
Write current	50 µA
Read latency	10 ns
Write latency	10 ns
Endurance	10¹⁵ cycles (unlimited for practical purposes)
Retention	10 years at 85 °C
Voltage	0.8 V
Why MRAM for T0: it is the only non-volatile technology with DRAM-class latency. It is used for OS boot, filesystem journal, and hot metadata — anything that must survive power loss but cannot tolerate NAND latency.

4.2 T1 — ReRAM (OxRAM)
Structure: a metal-insulator-metal stack (e.g., TiN/HfO₂/TiN). A filament forms or breaks in the oxide, changing resistance.

Parameter	Value
Cell size	4F² (at 2nm: ~13 nm²)
Read voltage	0.3 V
Write voltage	1.5 V (set), −1.5 V (reset)
Read latency	100 ns
Write latency	1 µs
Endurance	10⁹ cycles
Retention	10 years at 85 °C
Multi-level	2 bits/cell (4 resistance states)
Why ReRAM for T1: it is 10× denser than MRAM and 100× faster than NAND. It is the workhorse tier for active user data (game installs, documents, browser cache).

4.3 T2 — 3D NAND (QLC)
Structure: vertical NAND string (like Samsung V-NAND), 128 layers of charge-trap cells per die.

Parameter	Value
Cell size	~100 nm² effective (3D stacked)
Page size	16 KB
Block size	4 MB (256 pages)
Read latency	50 µs
Write (program) latency	500 µs
Erase latency	5 ms
Endurance	1,000 P/E cycles (QLC)
Retention	1 year at 85 °C (QLC)
Bits/cell	4 (QLC)
Why NAND for T2: it is the cheapest per bit by 10×. It is used only for cold data (archives, backups, video files) where 50 µs read latency is acceptable.

5. The Hardware FTL (Coordinate Translation)
5.1 The Software FTL Problem
Standard SSDs run an FTL on an embedded ARM core. The FTL maps a logical block address (LBA) to a physical NAND page. This mapping table is 4 bytes per 4K block:

4 TB drive = 1 billion blocks × 4 bytes = 4 GB mapping table.

The table lives in DRAM. Every I/O requires a DRAM lookup (80 ns) plus NAND access.

Garbage collection (GC) and wear leveling run in firmware, adding jitter.

5.2 The PSSD Hardware FTL
The PSSD replaces the software FTL with a hardware coordinate translator in the PMPW bridge controller:

Function	Implementation
LBA → physical coordinate	Content-addressable memory (CAM) with 1 M entries (one per 4 MB region)
Lookup latency	1 cycle (166 ps)
Mapping granularity	4 MB region (not 4 KB) — reduces table to 1 M entries
Within-region offset	Direct bit-address (no lookup)
GC	Hardware state machine, no firmware
Wear leveling	Round-robin counter per block, hardware
Why 4 MB granularity? A 4 TB drive with 4 KB granularity needs 1 billion entries. With 4 MB granularity, it needs 1 million — fitting in a 16 MB CAM in the bridge controller. The within-region offset is computed by simple bit-slicing (no lookup).

5.3 The Coordinate Format
A PSSD address is 48 bits:

text
[47:40] Tier (8 bits, 0–2)
[39:32] Layer (8 bits, 0–31)
[31:16] Row (16 bits, 0–65535)
[15:4]  Column offset (12 bits, 0–4095)
[3:0]   Byte offset within 16-byte word (4 bits)
This is a physical coordinate, not a packet. The CPU asserts it on the address TSVs, and the target cell responds. No queue, no arbitration (STDM schedule handles contention).

6. Read / Write / Erase Operations
6.1 Read Operation (T0 — MRAM)
CPU asserts 48-bit coordinate on address TSVs.

Bridge controller decodes tier = 0, layer = L, row = R, column = C.

Layer L's row decoder asserts row R.

Column C's MTJ is sensed by a 10 µA current source.

Sense amp compares against reference MTJ.

Data driven back up the 4,096 data TSVs.

Latency: 10 ns (cell) + 1 ns (decode) + 0.2 ns (bridge) = ~11.2 ns.

6.2 Read Operation (T1 — ReRAM)
Identical, but the sense amp measures resistance ratio (high/low) with a 0.3 V read.

Latency: 100 ns + 1 ns + 0.2 ns = ~101 ns.

6.3 Read Operation (T2 — NAND)
NAND is page-based, not bit-random:

CPU asserts coordinate with tier = 2.

Bridge controller identifies the 16 KB page.

NAND row decoder asserts the wordline.

All 16 KB of cells in the page are sensed simultaneously (one sense amp per column).

Page buffer latches the 16 KB.

Data streamed out over 4,096 TSVs (16 KB / 512 B/cycle = 32 cycles).

Latency: 50 µs (page read) + 5 ns (stream) = ~50 µs.

6.4 Write Operation (T0 — MRAM)
CPU asserts coordinate + data on TSVs.

Bridge controller drives 50 µA through the MTJ in the correct direction (parallel = 0, anti-parallel = 1).

MTJ switches in ~10 ns.

Latency: ~11 ns.

6.5 Write Operation (T1 — ReRAM)
CPU asserts coordinate + data.

Bridge controller applies 1.5 V (set) or −1.5 V (reset) across the oxide.

Filament forms/breaks in ~1 µs.

Latency: ~1 µs.

6.6 Write Operation (T2 — NAND)
NAND requires erase-before-write at the block level:

If the target page is not erased, the entire 4 MB block must be erased first (5 ms).

Then the 16 KB page is programmed with 500 µs of incremental step pulse programming (ISPP).

The page buffer holds the data during programming.

Latency: 500 µs (program) or 5.5 ms (erase + program).

This is why NAND is T2. The P-Controller (see §9) ensures that NAND is only written in large sequential bursts, amortizing the erase cost.

6.7 The Erase Problem in a Cross-Point Array
A critical hole in the original PSSD concept: NAND erase is a block-level operation. In a cross-point array where every cell shares TSVs, you cannot erase one block without affecting its neighbors.

Fix: the PSSD NAND tier uses isolated NAND strings with a dedicated source-line (SL) TSV per block. The SL TSV is separate from the data TSVs. When a block is erased, the SL is driven to 20 V while the data TSVs are floated. The block's cells tunnel-erase; neighboring blocks are isolated by the SL separation.

SL TSV count: 1 per 4 MB block × 65,536 blocks = 65,536 SL TSVs. That is 16× the data TSV count — physically impossible.

Better fix: NAND blocks are erased in batches. The PSSD has 64 SL TSVs, each shared by 1,024 blocks. A batch erase takes 5 ms × 1,024 = 5.1 s per SL — too slow.

Final fix: the PSSD NAND tier uses sub-block erase with a local charge pump per 256 blocks. Each charge pump is ~0.1 mm². 256 pumps = 25.6 mm² per layer. This is the honest area cost of NAND in a cross-point array. It is 12% of the die — acceptable.

7. Bandwidth & Latency Math
7.1 Streaming Bandwidth (All Tiers)
text
4,096 TSVs × 6 GHz = 24.576 Tb/s = 3.07 TB/s
This is the bridge limit, shared by all tiers. The P-Controller arbitrates via the token ring (see PENTAL-CPU-1.md §3.2.2).

7.2 Per-Tier Effective Bandwidth
Tier	Access granularity	Latency	Effective BW (single stream)
T0 (MRAM)	512 B	11 ns	512 B / 11 ns = 46 GB/s
T1 (ReRAM)	512 B	101 ns	512 B / 101 ns = 5 GB/s
T2 (NAND)	16 KB	50 µs	16 KB / 50 µs = 0.33 GB/s
But these are single-stream numbers. With 4,096 TSVs operating in parallel (one stream per TSV), the aggregate is:

Tier	Streams	Aggregate BW
T0	4,096	4,096 × 46 GB/s = 188 TB/s (limited by bridge to 3.07 TB/s)
T1	4,096	4,096 × 5 GB/s = 20 TB/s (limited by bridge to 3.07 TB/s)
T2	256	256 × 0.33 GB/s = 84 GB/s (NAND-limited)
The bridge is the bottleneck for T0 and T1. NAND is the bottleneck for T2.

7.3 Random 4K Read IOPS
Tier	IOPS
T0 (MRAM)	4,096 TSVs / 11 ns = 372 G IOPS (bridge-limited to ~3.07 TB/s / 4 KB = 750 M IOPS)
T1 (ReRAM)	4,096 / 101 ns = 40 G IOPS (bridge-limited to 750 M IOPS)
T2 (NAND)	256 / 50 µs = 5.1 M IOPS
Comparison: a high-end NVMe SSD does ~1.5 M IOPS. The PSSD T0 tier does 750 M IOPS — 500× faster.

8. Wear Leveling (Hardware)
NAND has limited P/E cycles (1,000 for QLC). Wear leveling must be hardware, not firmware.

8.1 The Counter Array
Each 4 MB block has a 16-bit P/E counter. 65,536 blocks × 2 bytes = 128 KB counter array in the bridge controller.

8.2 The Round-Robin Algorithm
When a write arrives:

Hardware FTL looks up the logical region's current physical block.

If the block has been written, the counter increments.

If the counter exceeds a threshold (e.g., 900 for QLC), the block is marked "worn."

A free block with the lowest counter is selected.

Data is copied (GC), and the mapping is updated.

Cost: 1 cycle per write for counter increment, 64 cycles for block remap. Amortized: < 0.1% overhead.

8.3 Tier-Level Wear Leveling
MRAM and ReRAM have effectively unlimited endurance (10¹⁵ and 10⁹). Only NAND needs wear leveling. The P-Controller limits NAND writes to sequential bursts (see §9), reducing P/E cycles by 10×.

9. P-Controller Integration (Law L8 — Hot/Cold Migration)
The P-Controller (see P-controller.md) monitors access frequency per 4 MB region:

Access frequency	Tier assignment
> 10⁶ accesses/hour	T0 (MRAM)
10³–10⁶ accesses/hour	T1 (ReRAM)
< 10³ accesses/hour	T2 (NAND)
9.1 Migration Policy
Every 1 second, the P-Controller samples the access counters:

If a T1 region exceeds 10⁶ accesses/hour, migrate to T0.

If a T2 region exceeds 10³ accesses/hour, migrate to T1.

If a T0 region drops below 10⁶ accesses/hour, migrate to T1.

If a T1 region drops below 10³ accesses/hour, migrate to T2.

Migration cost: copy 4 MB between tiers. T0→T1: 4 MB / 5 GB/s = 800 µs. T1→T2: 4 MB / 0.33 GB/s = 12 ms.

Migration is only worthwhile if the region will stay in the new tier for > 1 hour.

9.2 STET/DTET for Storage
STET (Same Task Exact Time): bulk sequential write (video recording, file copy). All 4,096 TSVs write the same pattern to consecutive addresses. Bandwidth: 3.07 TB/s.

DTET (Different Tasks Exact Time): random 4K reads (database, OS boot). Each TSV handles a different LBA. IOPS: 750 M.

The P-Controller classifies the workload and switches modes. Switch cost: ~100 ns.

10. ECC & Reliability
10.1 Error Rates
Tier	Raw BER	ECC needed
T0 (MRAM)	10⁻¹²	SECDED (single-error correct, double-error detect)
T1 (ReRAM)	10⁻⁹	SECDED + scrubbing
T2 (NAND)	10⁻³ (QLC, after cycling)	LDPC (low-density parity check)
10.2 ECC Overhead
Tier	ECC scheme	Overhead
T0	SECDED (8 bits per 64)	12.5%
T1	SECDED + scrub	12.5%
T2	LDPC (soft-decision)	~8% (but requires 2 reads for soft decode)
10.3 Scrubbing
T0: scrub every 24 hours (MRAM is robust).

T1: scrub every 6 hours (ReRAM can drift).

T2: scrub every 1 hour (QLC retention is poor).

Scrub bandwidth cost: 4.6 TB / 1 hour = 1.3 GB/s = 0.04% of bridge bandwidth. Negligible.

10.4 Power-Loss Protection
The PSSD has a 3 mF capacitor bank on the bridge controller. On power loss:

Capacitor provides 5 ms of hold-up power.

Bridge controller flushes all T0 and T1 write buffers.

NAND writes in progress are completed (500 µs is within the 5 ms window).

A "dirty shutdown" flag is written to T0.

Cost: 3 mF at 12 V = 216 mJ. Capacitor size: ~10 mm³. Acceptable.

11. Power Management
Tier	Idle power	Active power	Per-layer
T0 (MRAM)	0.1 W	2 W	4 layers = 8 W
T1 (ReRAM)	0.05 W	3 W	16 layers = 48 W
T2 (NAND)	0.01 W	1 W (read) / 5 W (program)	12 layers = 60 W
Total	~1 W	~120 W	—
Idle power is dominated by refresh-like leakage. MRAM and ReRAM have no refresh; NAND has no refresh but has charge leakage. The 1 W idle is the bridge controller + TSV drivers.

12. Comparison to NVMe SSDs
Metric	Samsung 990 Pro (NVMe)	PSSD Rev 1.0
Interface	PCIe 4.0 ×4	PMPW bridge
Protocol	NVMe (packetized)	Spatial coordinate
Capacity	4 TB	4.6 TB
Sequential read	7.5 GB/s	3.07 TB/s (T0/T1)
Sequential write	6.9 GB/s	3.07 TB/s (T0/T1)
Random 4K read	1.4 M IOPS	750 M IOPS (T0)
Random 4K write	1.3 M IOPS	750 M IOPS (T0)
Read latency	80 µs	11 ns (T0), 101 ns (T1), 50 µs (T2)
Write latency	200 µs	11 ns (T0), 1 µs (T1), 500 µs (T2)
Endurance	1,200 TBW	T0: unlimited, T1: 10⁹ cycles, T2: 1,000 P/E
Power	7 W	120 W active, 1 W idle
Form factor	M.2 2280	24×24×0.6 mm package
Cost	$300	~$8,000 (est.)
The PSSD is 400× faster on latency and 500× faster on IOPS. It costs 25× more. It is targeted at HFT, defense, and AI training — not consumer laptops.

13. The 32 TB Extension (73-Layer Supercomputer)
For the 73-layer PENTAL supercomputer, the PSSD scales to:

Parameter	4.6 TB (desktop)	32 TB (supercomputer)
T0 layers	4	8
T1 layers	16	32
T2 layers	12	64
Total layers	32	104
Stack height	400 µm	1.3 mm
Bridge TSVs	4,096	4,096 (same)
Aggregate BW	3.07 TB/s	3.07 TB/s (bridge-limited)
T2 BW	84 GB/s	168 GB/s
Power	120 W	400 W
Cooling	Air	Liquid mandatory
The bridge bandwidth does not scale — it is fixed at 3.07 TB/s by the 4,096 TSVs. To scale beyond 32 TB, the PSSD needs a second bridge (8,192 TSVs) or a photonic interconnect.

14. The Honest Limitations
Following the audit's discipline, here is what the PSSD cannot do:

NAND erase is still slow. 5 ms per block is physics. The PSSD hides it with large sequential writes, but random 4K writes to cold data still pay the erase cost.

The bridge is the bottleneck. 3.07 TB/s is shared by all tiers. T0 and T1 cannot both run at full speed simultaneously.

T2 is not a "fast" tier. 50 µs read latency is still 50 µs. The PSSD is not "all fast" — it is tiered, and cold data is slow.

Cost. MRAM and ReRAM are 10–50× more expensive per bit than NAND. The 4.6 TB PSSD costs ~$8,000, not $300.

Power. 120 W active is 17× an NVMe SSD. The T2 NAND program power (5 W/layer × 12) dominates.

The 4 MB FTL granularity means small files (< 4 MB) are stored as full regions, wasting space. A 1 KB file consumes 4 MB. Effective capacity for small files is 0.025%. This is the price of hardware FTL.

Fix for #6: the PSSD uses sub-region allocation for files < 4 MB. A 4 MB region is subdivided into 4,096 × 1 KB sub-blocks. The hardware FTL tracks sub-block allocation with a 1-bit valid flag per sub-block. This adds 512 KB of flags per 4 MB region — 0.0125% overhead. Acceptable.

15. Summary
The PSSD is the storage tier of the Pental System. It extends the PMPW grid into non-volatile memory, replacing the packetized NVMe queue with a spatial coordinate system. It is tiered (MRAM → ReRAM → NAND) to balance latency, density, and cost. It is orchestrated by the P-Controller, which migrates hot data up and cold data down (Law L8). It is 400× faster than NVMe on latency, 500× faster on IOPS, and 25× more expensive.

It is not a consumer SSD. It is a defense/HFT/AI-training storage engine.

End of PSSD.md — Revision 1.0