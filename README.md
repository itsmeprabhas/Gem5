# Gem5 Architecture Simulator: Cache Profiling & NVM Emulation

##  Overview
A computer architecture analysis project built on the **Gem5 Simulator**. This repository demonstrates low-level systems programming by modifying the underlying C++ Ruby Cache structures to track localized instruction and data cache hit/miss rates. Additionally, it features custom Python configuration models to emulate the latency and bandwidth constraints of Non-Volatile Memory (ReRAM NV3). The system was aggressively benchmarked using the **SPEC 2017 DeepSjeng** workload over a 200-billion tick execution window to measure exact memory stall penalties.

##  Technical Highlights
* **C++ Cache Profiling:** Injected custom tracking and mathematical logic into the Ruby Cache Engine (`CacheMemory.cc`) to monitor specific execution windows (3.5B to 4.0B ticks) and calculate hit/miss percentages in real-time while avoiding division-by-zero faults.
* **Python Memory Emulation:** Developed custom memory profiles within `DRAMInterface.py` to emulate ReRAM NV3 behavior. Successfully overrode baseline HBM timings to enforce a 4x Read Latency penalty, 15x Write Latency penalty, and 0.5x Bandwidth constraint.
* **Workload Benchmarking:** Executed massive multi-billion cycle simulations utilizing the SPEC 2017 `531.deepsjeng_r` (Chess AI) benchmark.

##  Key Findings
**1. Baseline L1/L2 Cache Analysis (HBM DRAM)**
* **L1-D Miss Rate:** Evaluated at **12.5%** (40,858 Hits | 5,837 Misses) during the observation window.
* **L2 Cache Thrashing:** Recorded **0 L2 Hits** and **5,838 L2 Misses**. Diagnosed as severe capacity misses caused by DeepSjeng utilizing a massive 720 MB hash table that entirely bypassed the 2 MB L2 cache capacity.

**2. Non-Volatile Memory (ReRAM) Penalty**
* **Baseline (DRAM):** 149,301,712,000 execution cycles
* **Emulated (ReRAM):** 214,022,151,000 execution cycles
* **Conclusion:** The system suffered an approximate **43% performance degradation**. Because the L2 cache was entirely bypassed, the CPU pipeline was forced to absorb the full, heavy latency penalties of the slow NVM main memory, resulting in constant processor stalls.

##  Repository Structure
* `/src/` — Modified C++ Ruby Cache engine structures (`CacheMemory.cc`, `CacheMemory.hh`).
* `/configs/` — Python-based memory interface configurations (`DRAMInterface.py`).
* `/results/` — Terminal execution logs and cycle counts for Baseline DRAM vs. ReRAM.
* `/docs/` — Final architectural analysis report detailing hit/miss rates and execution penalties.

##  Build & Run Instructions
**1. Recompile the Gem5 Simulator**
Because core Python and C++ files have been modified, you must recompile the X86 Gem5 executable using `scons`:
```bash
scons build/X86/gem5.opt -j 20
```
2. Run the Baseline Simulation (DRAM)
```bash
build/X86/gem5.opt configs/deprecated/example/se.py --cmd=<path_to_deepsjeng> --ruby --l1i_size=8kB --l1i_assoc=2 --l1d_size=16kB --l1d_assoc=2 --l2_size=2MB --l2_assoc=4 --mem-type=HBM_1000_4H_1x64 --fast-forward=1000000 --maxinsts=10000000
```
3. Run the NVM Simulation (ReRAM)
```bash
build/X86/gem5.opt configs/deprecated/example/se.py --cmd=<path_to_deepsjeng> --ruby --l1i_size=8kB --l1i_assoc=2 --l1d_size=16kB --l1d_assoc=2 --l2_size=2MB --l2_assoc=4 --mem-type=NVM_ReRAM --fast-forward=1000000 --maxinsts=10000000
```
**Tech Stack**
Languages: C++, Python, Bash/Shell
Tools & Frameworks: Gem5 Simulator, SPEC 2017 Benchmarks, Linux, Git
Concepts: Computer Architecture, Memory Hierarchies (L1/L2), Non-Volatile Memory (NVM), Performance Profiling
