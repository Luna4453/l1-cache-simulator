# Project Specification: Interactive L1 Cache Simulator & Visualizer

## 1. Project Purpose and Scope

### 1.1 Purpose
This project is an interactive, client-side web application that illustrates how a CPU L1 data cache operates. It allows users to configure cache geometry, execute memory access traces step-by-step, and observe how 32-bit memory addresses are decoded into **Tag**, **Index**, and **Block Offset** fields, how hits, misses, and LRU evictions occur, and how cache parameters affect performance metrics.

### 1.2 Deliverables
1. **Static Web Application:** Deployed publicly via GitHub Pages (with the live URL provided in `README.md`).
2. **Source Repository:** Hosted on GitHub (shared with `srivoire`) and organized with separate directories for source code (`src/`), tests (`tests/`), build output (`dist/`), and AI agent prompts/specs (`prompts/`).
3. **Documentation:** A `README.md` covering setup, build/test commands, live deployment URL, architecture decisions, and prompt files used.
4. **Automated Test Suite:** Unit and integration tests verifying all cache logic, address math, and edge cases.

### 1.3 Non-Goals and Exclusions
To keep the scope focused and accurate, this application will **NOT** implement:
- Multi-level cache hierarchies (L2/L3) or multi-core cache coherence protocols (e.g., MESI).
- Virtual memory, page tables, or TLBs (all addresses are treated as 32-bit physical addresses).
- Alternative replacement policies beyond LRU (such as FIFO or Random).
- Cycle-accurate pipeline timing or bus contention modeling (metrics are limited to hits, misses, hit rate, and memory write-backs).
- Backend servers, databases, or user authentication.

---

## 2. Requirements and Expected Behavior

### 2.1 Cache Configuration (Required)
Users must be able to configure the cache before starting or resetting a simulation:
- **Address Space:** Fixed 32-bit byte-addressable memory.
- **Cache Size (Bytes):** Power of 2, selectable from `{64, 128, 256, 512, 1024}` (default: `256`).
- **Block Size (Bytes):** Power of 2, selectable from `{4, 8, 16, 32}` (default: `16`).
- **Associativity:** Selectable from `{1 (Direct-Mapped), 2 (2-Way), 4 (4-Way), Fully Associative}` (default: `2`).
- **Replacement Policy:** Fixed to **LRU (Least Recently Used)** for all set-associative and fully associative configurations.
- **Write Policy:** Selectable from `{Write-Through (No-Write-Allocate), Write-Back (Write-Allocate)}` (default: `Write-Back`).

**Validation Rule:** Changing any configuration parameter resets the current simulation state. If a selected combination results in `Number of Sets < 1` (i.e., `Block Size * Associativity > Cache Size`), the UI must prevent the invalid state or display an inline validation error and disable execution until resolved.

### 2.2 Memory Trace Input (Required)
Users must be able to supply memory accesses in two ways:
1. **Manual / Trace Editor:** A text area accepting one access per line in the format:
   `<R|W> <hex_address>` (case-insensitive, e.g., `R 0x00000010` or `W 0x1A4`).
2. **Built-in Educational Presets:** At least 3 one-click preset traces that demonstrate core architecture concepts:
   - *Spatial Locality:* Sequential word accesses (e.g., `0x00, 0x04, 0x08, 0x0C, 0x10...`).
   - *Temporal Locality:* Repeated accesses to a small working set of addresses.
   - *Conflict Misses (Thrashing):* Strided accesses that map to the same set in a direct-mapped cache, illustrating how increasing associativity resolves conflict misses.

### 2.3 Execution Controls (Required)
- **Step Forward:** Executes the next memory access in the trace and updates all visual panels.
- **Run All:** Executes all remaining accesses in the trace to completion.
- **Reset:** Clears all cache lines (sets valid bits to `0`), resets statistics to `0`, and returns the trace pointer to line 1.

### 2.4 Visual Output & System Behavior (Required)
1. **Address Breakdown Panel:** Displays the current 32-bit address in hex and binary, color-coded into:
   - `Tag Bits` = 32 - Index Bits - Offset Bits
   - `Index Bits` = log2(Number of Sets) (0 bits for Fully Associative)
   - `Offset Bits` = log2(Block Size)
2. **Cache Table Visualization:** Displays every Set and Way with:
   - `Valid` bit (`0` or `1`)
   - `Dirty` bit (`0` or `1`, shown when Write-Back is active)
   - `Tag` (in hexadecimal)
   - `LRU Rank` (indicating recency order within the set so users can see which block will be evicted next)
   - Visual highlight on the active set/way indicating **Hit** (green), **Miss/Fill** (yellow), or **Eviction** (red).
3. **Access Log & Metrics Panel:**
   - Displays total accesses, Hits, Misses, Hit Rate (%), Miss Rate (%), Evictions, and Memory Writes (write-throughs + dirty write-backs).
   - Displays a scrollable log of executed accesses with their outcome (`HIT`, `MISS`, `MISS + EVICT`, `MISS + WRITE-BACK`).

### 2.5 Optional Features (Nice-to-Have)
*The agent should NOT implement these until all Required features and tests pass:*
- **Step Backward:** Undo the previous memory access.
- **3Cs Miss Classification:** Categorizing misses into Compulsory, Capacity, and Conflict misses.
- **Export Trace/Stats:** Download the execution log as a `.csv` file.

---

## 3. Technical Details and Constraints

### 3.1 Tech Stack
- **Language:** TypeScript (strict mode enabled)
- **Framework:** React 18+ with Vite (for fast static bundling)
- **Styling:** Tailwind CSS or plain modular CSS (agent's choice)
- **Testing:** Vitest (unit and integration tests)
- **Runtime/Build:** Node.js 18+ / npm

### 3.2 Architecture & Directory Structure
The simulation engine **must** be strictly decoupled from the React UI so it can be unit-tested independently without a DOM:
```text
/
├── prompts/          # spec.md and any agent prompt/instruction files
├── src/
│   ├── engine/       # Pure TypeScript cache simulator logic (zero React imports)
│   ├── components/   # React UI components
│   ├── presets/      # Built-in memory trace presets
│   └── main.tsx
├── tests/            # Vitest test suites targeting src/engine/
├── dist/             # Production static build (configured with relative base path './')
└── README.md
