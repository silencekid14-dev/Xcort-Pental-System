Pental-RAM (PRAM) — Technical Architecture Specification
Revision 2.0 — Extended Engineering Edition
1. Core Philosophy
PRAM replaces the packetized memory bus with a spatial coordinate system. An address is a physical voltage coordinate, not a request packet. Revision 2.0 fixes the layer-select hole, adds the refresh schedule, the write RC model, the sense-amp budget, ECC, and realistic die sizes.

2. Physical Layer Stacking
2.1 Vertical TSV Array
4,096 data TSVs at 15 µm pitch.

Dedicated, non-multiplexed, running through all 8 layers.

Hybrid-bonded (TSMC SoIC) to the CPU above.

2.2 Horizontal Lines
X row lines: 4,096 per layer (wordlines).

Y bank-select lines: 1,024 per layer (mandatory, see §3).

Pitch: 50 nm.

2.3 Memory Cells
1T1C DRAM cell at each (TSV, row, bank) intersection. 20 fF capacitor, 2–5 kΩ pass transistor.

3. The Layer-Select Fix (Previously a Fatal Hole)
The audit's PRAM hole: a TSV runs through all 8 layers, so asserting row #512 on Layer 2 also turns on the cell at row #512 on Layer 5. The original spec quietly added a bank-select line only in the 16 GB section — Revision 2.0 makes it mandatory in the base design.

3.1 Corrected Cell Topology
Each layer has a Layer-Select (LS) line. The pass transistor's gate is driven by a 3-input AND:

text
gate = Row_Select AND Bank_Select AND Layer_Select
Layer_Select is a 1-bit vertical control line per layer, driven by the CPU via a dedicated TSV.

Only one layer's LS is high at a time.

The other 7 layers' transistors are off; their capacitors do not dump onto the TSV.

3.2 Cost
1 extra vertical control TSV per layer = 8 control TSVs total.

1 extra AND gate per cell = 128 G gates. At 2nm (6T/mm²), that's ~21 mm² — 17% die overhead.

This is the price of correctness. The original spec was wrong; Revision 2.0 pays the area cost.

4. Read Operation — The "Direct Sense" Path
CPU drives target TSV to 0.6 V precharge.

CPU asserts Layer_Select, Bank_Select, Row_Select.

Pass transistor opens; capacitor charge redistributes onto TSV.

Sense amp detects 50–100 mV swing.

Sense amp drives full CMOS level back up the TSV.

4.1 Latency Budget (16 GB, 8-layer)
Component	Time
TSV propagation (up + down, 10 µm × 8 layers)	0.4 ns
Bank/row decode	1.8 ns
Sense amp settle	0.8 ns
Total	~3.0 ns
5. Write Operation — RC Model (Previously Hand-Waved)
The original claimed 2.8 ns. Revision 2.0 derives it:

5.1 RC Calculation
Component	Value
TSV capacitance	8 pF (500 µm × 1–2 fF/µm × 8 layers)
Pass transistor R_on	5 kΩ
Driver R_out	100 Ω
RC time constant	(5k + 100) × 8 pF = 40.8 ns
Wait — that contradicts the 2.8 ns claim. The fix: the CPU does not drive through the pass transistor. It drives the TSV directly from a low-impedance output driver (100 Ω), and the pass transistor is only for latching the capacitor after the TSV is stable.

Corrected write	Time
Driver charges TSV (100 Ω × 8 pF)	0.8 ns
Pass transistor latches capacitor	0.3 ns
Layer/bank/row decode	1.8 ns
Total	~2.9 ns
This is physically real, but only because the driver is 100 Ω. 4,096 such drivers at 100 Ω each is a significant analog area cost (~4 mm²).

6. Sense Amplifier Design (Previously Missing)
Parameter	Value
Offset (matched)	< 5 mV
Settle time	0.8 ns
Area per amp	50 µm²
Power per amp	20 µW
Total (4,096 amps)	0.2 mm², 82 mW
Sense amps are ~1% of die area, not 30–40% like standard DRAM — because PRAM has one amp per TSV, not one per bitline column.

7. Bandwidth & Throughput — Streaming vs. Random (Previously Conflated)
7.1 Streaming Bandwidth
text
4,096 bits/cycle × 6 GHz = 24.576 Tb/s = 3.07 TB/s
This is real only when all 4,096 TSVs carry useful data in the same cycle.

7.2 Random Transaction Rate
Latency: 4 ns.

Transactions in flight: 4 ns ÷ 166 ps = 24.

Each transaction = 512 bits (one cache line).

Random throughput = 24 × 512 bits / 4 ns = 3.07 Tb/s = 384 GB/s.

This is the honest random-access number. It is 8× lower than the streaming number, but still 5× DDR5's random throughput.

8. Refresh Mechanism (Previously Claimed "0%")
1T1C leaks. At 2nm, 20 fF loses 10% charge in ~64 ms. Every cell must be refreshed.

8.1 Refresh Cost
Parameter	Value
Total bits	128 Gb
Refresh period	64 ms
Data to refresh per cycle	128 Gb / 64 ms = 2 Tb/s
Available bandwidth	3.07 TB/s = 24.576 Tb/s
Refresh overhead	8.1%
The original "0% via idle-lane scavenging" is wrong. Revision 2.0 states the honest number: 8% of bandwidth is consumed by refresh, leaving 2.83 TB/s for compute.

8.2 Refresh Scheduling
Distributed refresh: 1/64th of rows refreshed every 1 ms.

Refreshes are interleaved into the STDM schedule.

The 8% is a hard floor, not opportunistic.

9. 16 GB Extension (Banking)
Parameter	Value
Layers	8 (2 GB each)
TSVs	4,096
Banks/layer	1,024
Rows/bank	4,096
Capacity/bank	16 Mb
Horizontal wires/layer	5,120 (feasible)
Latency	~4.0 ns
Bandwidth	3.07 TB/s streaming, 384 GB/s random
10. 64 GB Extension
Parameter	16 GB	64 GB
Layers	8	32
Stack height	0.8 mm	3.2 mm
Read latency	4.0 ns	12–15 ns
Yield	~70%	<30%
Cost	~$10k	~$60k
Cooling	Air/vapor	Liquid mandatory
11. ECC & Reliability (Previously Missing)
11.1 Soft Error Rate
2nm DRAM SER: ~10⁻³ FIT/Mbit.

128 Gb = 128,000 Mbit → 128 FIT.

At sea level: ~1 error every 8 hours.

At altitude: ~1 error every 2.7 hours.

11.2 SECDED
SECDED ECC on every 64-bit word: +8 bits per 64 = 12.5% area overhead.

Scrubbing: every 24 hours, read-and-rewrite all lines to clear accumulated errors.

Spare rows/columns: 1% spare for hard-error repair.

Without ECC, 16 GB PRAM is unusable for the defense/HFT markets the spec names. Revision 2.0 makes ECC mandatory.

12. Realistic Die Sizes (Previously Wrong)
The original claimed 22×22 mm die for 16 Gb. That is 6–8× too large.

Parameter	Original claim	Corrected
16 Gb die	484 mm²	~70 mm²
8-layer stack	484 mm²	~70 mm² (stacked)
Package	22×22 mm	~22×22 mm (package, not die)
The 22×22 mm is the package size, not the die. Revision 2.0 corrects this.

13. PRAM vs. DRAM vs. SRAM — Honest Comparison
Metric	DDR5	SRAM (L3)	PRAM (16 GB)
Latency	80 ns	1.2 ns	4.0 ns
Streaming BW	80 GB/s	1 TB/s	3.07 TB/s
Random BW	40 GB/s	800 GB/s	384 GB/s
Capacity	64 GB	32 MB	16 GB
ECC	Yes	Yes	Yes (SECDED)
Refresh	5%	0%	8%
End of PRAM.md — Revision 2.0

