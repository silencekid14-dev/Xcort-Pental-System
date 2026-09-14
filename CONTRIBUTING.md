# Contributing to Pental-System

Thank you for your interest in the Pental-System architecture. We welcome contributions from hardware architects, thermal engineers, compiler designers, and semiconductor experts who want to help advance this 3D-stacked computing blueprint.

Because this project is a physically grounded computing skyscraper, all contributions must adhere to strict engineering discipline and open-source licensing rules.

---

## 1. The Legal Framework: CERN-OHL-W v2

By contributing to this repository, you agree that your submissions will be licensed under the **CERN Open Hardware Licence Version 2 – Weakly Reciprocal (CERN-OHL-W v2)**. 

### What this means for your contributions:
* **Anti-Stealing Protection:** Any modifications, optimizations, or layout changes you submit to the Pental core specifications must remain completely open-source. No one—including corporations—can take your contributions and lock them away in a private, closed-source architecture patent.
* **Attribution:** Your name and contribution will be permanently preserved in the repository history, granting you credit as a co-architect of the system.
* **Downstream Freedom:** If anyone uses the specifications to build an RTL simulator (Verilog/VHDL) or fabricates a physical test chip via a silicon shuttle program, the derivative hardware design must also be released openly under these reciprocal terms.

---

## 2. Technical Submission Guidelines

The defining rule of the Pental-System repository is **Physics-Level Rigor**. Marketing language, theoretical buzzwords, or unproven assumptions will be rejected immediately.

### Core Architectural Requirements:
1. **The Seven Laws Invariant:** Every calculation, metric, and architectural block you propose must strictly align with the **Seven Laws of the Pental System** (L1 through L7, plus the L8 Storage Corollary). If your numbers contradict a Law, the proposal is wrong.
2. **Derivable Constants:** Any physics-level claim (thermal performance, wire resistance, latency, power draw) must be explicitly derived from real material constants (e.g., thermal conductivity of copper, properties of 2nm process logic, hybrid bonding alignment tolerances). 
3. **Honest Caveats:** If an architectural change introduces a trade-off (e.g., adding an interface layer that increases die area or adds latency cycles), you **must** call it out explicitly. We favor engineering correctness over perfect-looking marketing metrics.

---

## 3. How to Open an Issue or Propose a Fix

If you find a mathematical hole, a physics contradiction, or a bottleneck where the numbers do not close:

1. **Check Existing Issues:** Ensure no one else has already flagged the problem.
2. **Open a "Technical Audit" Issue:** Use the following template to format your issue:
   * **Target File & Section:** (e.g., `PRAM.md` §5.1)
   * **The Claim as Written:** Quote the exact text or formula.
   * **The Contradiction:** Detail the physical or mathematical conflict.
   * **Proposed Resolution:** Suggest an honest numerical adjustment or architectural fix.

---

## 4. Pull Request (PR) Process

1. **Fork the Repository:** Create a personal branch for your changes.
2. **Keep it Atomic:** Focus each PR on fixing a specific architectural hole or implementing a defined phase of the roadmap.
3. **Verify the Arithmetic:** Double-check your wire counts, latency cycles, and bandwidth math before submitting.
4. **Submit the PR:** Provide a clear summary explaining how your change fixes the targeted engineering bottleneck. 

All PRs require a technical review from the repository maintainers to verify physical feasibility before they are merged into the main branch.
