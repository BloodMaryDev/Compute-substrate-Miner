# Compute Substrate Forge

A high-performance **CUDA SHA-256d miner** for the **Compute Substrate (CSD)**
network. One binary mines every NVIDIA RTX 30/40/50-series GPU in your rig, from a
single process, with a live per-GPU dashboard.

- **Fast**: ~+5% faster kernel than the reference miner (hoisted SHA rounds +
  tuned occupancy).
- **Multi-GPU**: `--all-gpus` runs every card in one process, each on its own pool
  connection (distinct extranonce → no nonce collision, no duplicate work).
- **Per-card auto-tune** with a persistent cache, **NVML telemetry** (temp/power/
  efficiency), and a **configurable pool**.
  
---

## Which binary?

| Binary | GPUs | Driver |
|---|---|---|
| **`compute-substrate-forge`** | RTX **3000 / 4000 / 5000** | CUDA 12.8+ / 13.x |
| **`compute-substrate-forge-cuda124`** | RTX **3000 / 4000** | CUDA **12.4** |
| **`compute-substrate-forge-hiveos.tar.gz`** | 3000 / 4000 / 5000 (HiveOS) | — |

The main binary is **universal**: a `compute_75` kernel that the driver JIT-compiles
to each card's native code — same full speed on Blackwell as a native build.
Use the `cuda124` build only for rigs stuck on a CUDA 12.4 driver (no Blackwell).

---

## Quick start

```bash

# 1. Cuda >13.0 Linux (Series 5000/4000/3000) mine every GPU to your address
./compute-substrate-forge --all-gpus --address YOURADDRESS --pool pool.yamaduo.no:3333  --worker WORKERNAME --auto-tune

# 2. Cuda >12.4 Linux (Serie 4000/3000) mine every GPU to your address
./compute-substrate-forge-cuda124 --all-gpus --address YOURADDRESS --pool pool.yamaduo.no:3333  --worker WORKERNAME --auto-tune

# 3. HiveOS in the shell (Series 5000/4000/3000) mine every GPU to your address
./csd-gpu-miner --all-gpus --address YOURADDRESS --pool pool.yamaduo.no:3333  --worker WORKERNAME --auto-tune

```

Example Flighsheet Hive-OS 

<img width="673" height="696" alt="image" src="https://github.com/user-attachments/assets/87975bd0-f318-483d-bbb2-a50a74899acd" />

## Options (the ones you need)

| Flag | What |
|---|---|
| `--address <addr20>` | **required** — your payout address |
| `--all-gpus` | mine every GPU in one process (recommended for a rig) |
| `--device <n>` | mine a single GPU instead (index from `devices`) |
| `--pool <host:port>` | mine to a different pool (default: built-in) |
| `--worker <name>` | rig name for the pool dashboard (display-only) |
| `--auto-tune` | benchmark + tune each card's launch geometry (see below) |
| `--backend <auto\|cuda\|cpu>` | backend select (default `auto`) |
| `--stats-port <p>` | xmrig-style `/1/summary` JSON on localhost:p |
| `--config <file.toml>` | load settings from a TOML file |

Subcommands: `newwallet`, `selftest`, `devices`, `bench`. Advanced flags (geometry,
CPU, watchdog, thermal, …) still exist but are hidden from `--help` for clarity.

Config file (`config.toml`, or `--config`):
```toml
address = "your40hexaddr20"
pool    = "pool.yamaduo.no:3333"   # optional
worker  = "rig1"                   # optional
```

---

## Auto-tune (per card)

`--auto-tune` benchmarks each GPU's launch geometry and picks the fastest, **per
card**, and **persists each card's winner** to `~/.config/csd-pool-miner/
autotune.toml`. Run it once; later starts (without `--auto-tune`) reuse every
card's tuned geometry automatically. Recommended on first run:

```bash
./compute-substrate-forge --all-gpus --address YOURADDRESS --pool pool.yamaduo.no:3333  --worker WORKERNAME --auto-tune
```

---

## HiveOS

Use `compute-substrate-forge-hiveos.tar.gz` as a **custom miner**. Full turnkey
flight-sheet steps are in `FLIGHTSHEET.md` inside the tarball. In short:

1. Host the tarball at a direct URL (GitHub release, web server, …).
2. Flight sheet → Miner **Custom** → Installation URL = that URL,
   **Wallet template** = `%WAL%`, **Pool URL** = `pool.yamaduo.no:3333` (or empty),
   **Extra config arguments** = `--all-gpus`.
3. Apply to your rigs — HiveOS installs and runs it; stats show in the UI.

---

## Live dashboard

`--all-gpus` prints a refreshing per-GPU table (hashrate, temp, power, efficiency,
accepted/rejected shares per card + a rig total). Under it only **new jobs** and
**accepted/rejected shares** scroll. NVML telemetry (temp/power) is built in.

```
 GPU  Device                 Hashrate      Temp     Power     Shares
  0   RTX 5080               7.16 GH/s    63 C    224 W    a=3 r=0  eff 31.9 MH/s/W
  ...
 TOTAL                       30.56 GH/s           1207 W    a=13 r=0
```

---

## Performance notes

- SHA-256d is **current-limited (EDP)** on Blackwell: cards run below their power
  limit (e.g. ~270 W of a 360 W 5080) at ~3.0 GHz core — this is normal and
  efficient. Raising the power limit does **not** help; only a V/f-curve undervolt
  (needs `nvidia-settings` + X11) can push further.
- A few `low difficulty` share rejects at startup are normal (pool vardiff ramping).

---

Voluntary donations (separate, optional): `0x91723defb8e9dac838f0faf2ec6beec3282d1c3c`.

## License

See `LICENSE` / `LICENSE-APACHE` / `LICENSE-MIT` and `TRADEMARK.md`. This is opt-in
mining software you run on your own hardware; it mines only when you start it, to
your own payout address.
