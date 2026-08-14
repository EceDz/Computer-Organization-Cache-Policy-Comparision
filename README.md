# Computer Organization Cache Replacement Policy Comparison

A Computer Organization / Computer Architecture  project comparing L1 data cache replacement policies (**LRU**, **FIFO**, and **CLOCK / second-chance**) using [MacSim](https://github.com/gthparch/macsim), a cycle-level heterogeneous CPU-GPU architecture simulator. This repo contains the **modified MacSim source files and simulation config**, not a full MacSim installation. A full write-up and results analysis is included in [`Report.pdf`](Report.pdf).

## What Was Modified

The replacement-policy logic lives in `cache_c::find_replacement_line()` in [`macsim-src/cache.cc`](macsim-src/cache.cc), which selects between three policies at runtime based on existing MacSim knobs:

| Policy | Enabled by | Implementation |
|---|---|---|
| **LRU** (default) | Both knobs `false` | Original MacSim behavior — scans the set and evicts the entry with the oldest `m_last_access_time` |
| **FIFO** | `KNOB_WB_FIFO = true` | Evicts the entry with the oldest `m_fifo_insertion_time` (insertion order, not access order) |
| **CLOCK** (second-chance) | `KNOB_CACHE_USE_PSEUDO_LRU = true` | Walks the set with a per-set clock hand; entries with a set reference bit get a second chance (bit cleared) before being considered for eviction |

Supporting changes:
- `cache_entry_c` (in [`macsim-src/cache.h`](macsim-src/cache.h)) gains `m_reference_bit` (for CLOCK) and `m_fifo_insertion_time` (for FIFO) fields.
- `update_line_on_hit()` sets the CLOCK reference bit on a hit, and updates the LRU timestamp for all policies for consistency.
- [`macsim-src/macsim.cc`](macsim-src/macsim.cc)'s `finalize()` was extended to print a summary at the end of a run: replacement policy in effect, cache hit/miss counts and rates, IPC, and cycle/instruction totals.

## Requirements

This is **not a standalone build** — it's a set of modified files meant to be applied on top of a full [MacSim](https://github.com/gthparch/macsim) source tree (a large simulator with its own build system, dependencies, and trace-file infrastructure). To use this project you need:

- A working MacSim checkout, built per MacSim's own instructions
- The files here (`macsim-src/cache.cc`, `cache.h`, `macsim.cc`, `macsim.h`) placed over MacSim's corresponding source files
- A memory-access trace file compatible with MacSim's trace format

## Running

From a built MacSim binary directory, with `params.in` and `trace_file_list` (both provided here) in place:

```bash
./macsim
```

**Before running:** [`macsim-bin/trace_file_list`](macsim-bin/trace_file_list) points to a hardcoded local path (`/home/vboxuser/Desktop/Assignment3Materials/benchmark/trace.txt`) — update it to point at your own trace file before running.

To select a replacement policy, set the corresponding knob in [`macsim-bin/params.in`](macsim-bin/params.in) (or via the command line, per MacSim's knob-override convention):

- `cache_use_pseudo_lru 1` → CLOCK policy
- `wb_fifo 1` → FIFO policy
- neither set → LRU policy (default)

At the end of a run, the simulator prints a results block reporting the active policy, cache hit/miss statistics, and IPC — this is the primary data used for the cross-policy comparison in `Report.pdf`.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
