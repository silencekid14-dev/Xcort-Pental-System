P-controller.md — The Pental Orchestration Controller
Hardware-Level Idle Detection & Program-to-Layer Assignment
1. Executive Summary
The P-Controller is the missing orchestration layer of the Pental System. It is a hardware block (not an OS scheduler) that:

Continuously monitors ALU utilization on every layer.

Detects idle cores and idle layers.

Classifies running programs by their execution profile.

Assigns or migrates programs to any layer with free capacity.

Enforces the Law of Isolation (L3) and the Law of the P-Controller (L7).

It lives in Layer 1 (the PMPW dispatch layer) and communicates with all layers via the vertical TSV control wires.

2. Why It Exists
Standard OS schedulers see a flat 16-core CPU. They cannot see that:

Layer 4 is running Windows 11 and must never be preempted.

Layer 2 has 3 idle cores that could run a physics job.

Layer 3's ALUs are 90% utilized but its cores are only 40% utilized (SIMD lanes idle).

A program needs DTET (divergent shaders) but is scheduled on a STET layer.

The OS scheduler is layer-blind. The P-Controller is layer-aware.

3. Hardware Architecture
text
┌──────────────────────────────────────────────────────────────┐
│                     P-CONTROLLER (Layer 1)                   │
│                                                              │
│  ┌────────────────┐   ┌────────────────┐   ┌──────────────┐  │
│  │  Utilization   │   │  Program       │   │  Assignment  │  │
│  │  Monitor       │──▶│  Classifier    │──▶│  Engine      │  │
│  │  (per ALU)     │   │  (STET/DTET)   │   │  (migrate)   │  │
│  └────────────────┘   └────────────────┘   └──────────────┘  │
│         │                     │                    │         │
│         ▼                     ▼                    ▼         │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Program Table (256 entries)                         │    │
│  │  PID | layer | core_mask | mem_footprint | class     │    │
│  └──────────────────────────────────────────────────────┘    │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  PMPW Control Wires (to all 4 layers)                │    │
│  └──────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────┘
3.1 Utilization Monitor
Per-ALU counter: each ALU has a 1-bit "busy" line.

Per-layer aggregator: sums 4,096 busy bits → 12-bit utilization value.

Update rate: every 64 cycles.

Output: util[0..3] (one per layer), 0–100%.

3.2 Program Classifier
Classifies each running program into one of four classes:

Class	Signature	Target layer type
OS	Ring-0, high interrupt rate	Layer 4 (pinned)
STET	Uniform SIMD, >90% ALU busy	Any layer with free ALUs
DTET	Divergent branches, <60% ALU busy	Layer with fewest programs
IO	Blocked on PRAM/GPU	Layer 1 (dispatch)
3.3 Assignment Engine
Every 1,024 cycles (≈ 200 ns), the Assignment Engine:

Reads util[0..3].

Finds the layer with the lowest utilization.

Reads the Program Table.

If a DTET program is running on a high-utilization layer, considers migration.

If an idle layer exists (<20% util), assigns the highest-priority waiting program.

3.4 Migration Cost
Operation	Cost
Save program state (registers + PC)	4 KB
Transfer via PMPW to target layer	~200 ns
Restore state	4 KB
Flush L1/L2 on source	64 cycles
Total	~400 ns
Migration is only worthwhile if the program will run for >10 µs on the target layer.

4. The Program Table
256 entries, stored in Layer 1 SRAM. Each entry:

Field	Bits	Meaning
PID	16	Process ID
Layer	2	Current layer
Core mask	4	Which cores on that layer
Mem footprint	32	PRAM bytes
Class	2	OS/STET/DTET/IO
Priority	4	0–15
Pinned	1	Cannot migrate (e.g., OS)
5. Worked Example: Windows 11 + GTA + Minecraft
5.1 Initial Assignment
Program	Class	Layer	Reason
Windows 11	OS	Layer 4	Pinned (Ring 0, interrupts)
Discord	IO	Layer 4	Low compute, high IO
Minecraft	STET	Layer 3	Chunk math is uniform
GTA	DTET	Layer 2	Physics + scripts diverge
GPU dispatch	IO	Layer 1	Frame packing
5.2 Dynamic Reassignment
At T = 30 s, the P-Controller sees:

Layer 3 util = 45% (Minecraft paused, loading chunks).

Layer 2 util = 95% (GTA physics overload).

Layer 1 util = 15%.

Action: migrate GTA's script thread (a DTET sub-task) to Layer 1's idle cores. Migration cost: 400 ns. GTA's frame time drops from 22 ms to 16 ms.

5.3 Thermal Throttling Response
If Layer 2's temperature exceeds 95 °C:

P-Controller reduces Layer 2's clock via DVFS.

Migrates the lowest-priority DTET program to Layer 1.

If still hot, migrates Minecraft from Layer 3 to Layer 2 (spreading heat).

6. Fault Tolerance
If a layer dies (KGD test failure after bonding):

P-Controller marks the layer as DEAD in the Program Table.

Migrates all programs to surviving layers.

Reduces total ALU count in the utilization monitor.

Continues booting with 3/4 layers.

If a core dies within a layer:

P-Controller removes it from the core mask.

Redistributes its work to the other 3 cores on that layer.

Alerts the OS via an APIC interrupt.

7. OS Interface
The P-Controller exposes a memory-mapped register block to the OS:

Register	Address	Function
PCTL_UTIL[0..3]	0xFEE00000	Read layer utilization
PCTL_ASSIGN	0xFEE00010	Write PID → preferred layer
PCTL_MIGRATE	0xFEE00014	Force migration
PCTL_PIN	0xFEE00018	Pin a program to a layer
PCTL_STATUS	0xFEE0001C	Read program table
Windows 11's scheduler can read these registers to make better decisions, but the P-Controller acts autonomously even if the OS ignores them.

8. Interaction with the PGPU
The P-Controller also monitors the PGPU's 4 layers:

If PGPU Layer 2 is idle (no diffuse rays), it can lend ALUs to PENTAL-CPU Layer 3 for chunk math.

This is the cross-chip PMPW extension: the PGPU and CPU share the same vertical TSV bank.

9. The 73-Layer P-Controller
For the 73-layer supercomputer:

Program Table expands to 4,096 entries.

Utilization monitor: 73 × 4,096 ALUs = 299,008 busy bits.

Assignment engine runs every 4,096 cycles.

Migration uses the vertical TSV token ring (Law L2).

10. The Laws Satisfied
Law	How P-Controller satisfies it
L1 (Verticality)	Assigns work to stacked layers, not a flat pool
L2 (Matrix Wire)	Uses PMPW control wires, not packet queues
L3 (Isolation)	Pins OS to Layer 4; a game crash cannot preempt it
L4 (Thermal Escape)	Migrates work away from hot layers
L5 (Adaptive Coordination)	Classifies STET vs. DTET and assigns accordingly
L6 (Spatial Memory)	Uses PRAM coordinates for program state
L7 (P-Controller)	Is the law — no ALU idles if work exists
11. Summary
The P-Controller is the missing orchestration layer of the Pental System. Without it, the layers are isolated islands. With it, they form a single adaptive compute fabric that routes work to idle capacity, respects thermal limits, tolerates faults, and never lets an ALU idle if work exists anywhere in the stack.

End of P-controller.md — Revision 2.0