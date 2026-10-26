# SRC 1-Bus — Signal & Port Contract

This file is the interface contract between the two of us. Every module port and
every control signal is defined here. Module testbenches are written **from this
file only** (cross-verification), so if something is not written here, it does
not exist.

**Rule:** any change to a port, name, width or meaning goes through a PR on this
file *before* the code changes. Both partners approve.

Course references: "FS p.N" = course formula sheet page N; "Class N" = tutorial N.

---

## 1. Conventions

| Topic | Decision |
|-------|----------|
| Names | Course names, exactly as written (case-sensitive): `PCout`, `MAin`, `Gra`, `BAout`, `C2out`, `CONin`... |
| Renamed signals | `C=B` → `CeqB`, `n=0` → `n_eq_0` (illegal identifiers in Verilog). Nothing else is renamed. |
| Keywords | `AND`, `OR`, `NOT` (uppercase) are legal identifiers; Verilog primitives are lowercase. Never use the lowercase forms as signal names. |
| Polarity | All control and status signals are 1 bit, **active high**. |
| Clock | Single clock `clk`. All registers load on `posedge clk`. |
| Reset | `rst`, **synchronous, active high**. Effects listed per module. |
| Step timing | One step Ti = one clock cycle. Control signals are combinational outputs of `control_unit` and are stable for the whole cycle. Every `*in` signal loads on the rising edge that ends the step. |
| Bit numbering | `[31:0]`, bit 31 = MSB, matching course notation `<31..0>`. |
| Data bus | One 32-bit net named `bus`. Built as a one-hot mux in `bus.v` (no tri-state). |
| Bus value when idle | If no `*out` signal is active, `bus = 32'h0`. |
| Files | Every `.v` file starts with `` `default_nettype none `` and ends with `` `default_nettype wire ``. |

---

## 2. Instruction fields (FS p.8-9, Class 3)

| Field | Bits | Notes |
|-------|------|-------|
| `op` | IR<31..27> | opcode, 5 bits |
| `ra` | IR<26..22> | |
| `rb` | IR<21..17> | |
| `rc` | IR<16..12> | |
| `c1` | IR<21..0> | 22 bits, sign-extended to 32 by `C1out` |
| `c2` | IR<16..0> | 17 bits, sign-extended to 32 by `C2out` |
| `c3` | IR<11..0> | `cond` = IR<2..0> for branches; shift count = IR<4..0> |

---

## 3. Control signals (driven by `control_unit`)

| Signal | Used by | Meaning (course RTN) | Ref |
|--------|---------|----------------------|-----|
| `Gra` | regfile | select R[ra] | FS p.3, Class 5 |
| `Grb` | regfile | select R[rb] | FS p.3, Class 5 |
| `Grc` | regfile | select R[rc] | FS p.3, Class 5 |
| `Rin` | regfile | R[selected] ← bus | FS p.3 |
| `Rout` | regfile, bus | bus ← R[selected] | FS p.3 |
| `BAout` | regfile, bus | bus ← (selected = R0 ? 0 : R[selected]) | FS p.3 |
| `PCin` | datapath (PC) | PC ← bus | Class 5 |
| `PCout` | bus | bus ← PC | Class 5 |
| `IRin` | ir | IR ← bus | FS p.4 |
| `C1out` | bus | bus ← c1 (sign-extended) | FS p.4 |
| `C2out` | bus | bus ← c2 (sign-extended) | FS p.4 |
| `MAin` | mem_if | MA ← bus | FS p.4 |
| `MDbus` | mem_if | MD ← bus | FS p.4 |
| `MDout` | bus | bus ← MD | FS p.4 |
| `Read` | mem_if, memory | MD ← M[MA] | FS p.4, Class 5 |
| `Write` | memory | M[MA] ← MD | FS p.4, Class 5 |
| `Wait` | control_unit (internal) | hold the step counter until `Done` | FS p.4, Class 5 |
| `Ain` | alu (A reg) | A ← bus | FS p.5 |
| `Cin` | alu (C reg) | C ← ALU result | FS p.5 |
| `Cout` | bus | bus ← C | FS p.5 |
| `ADD` | alu | C ← A + bus | FS p.5 |
| `SUB` | alu | C ← A − bus | FS p.5 |
| `AND` | alu | C ← A ∧ bus | FS p.5 |
| `OR` | alu | C ← A ∨ bus | FS p.5 |
| `NOT` | alu | C ← ¬bus | FS p.5 |
| `CeqB` | alu | C ← bus | FS p.5 |
| `INC4` | alu | C ← bus + 4 | FS p.5 |
| `SHR` | alu | C ← bus shifted right 1, 0 into MSB | FS p.5 |
| `SHRA` | alu | C ← bus shifted right 1, sign kept | FS p.5 |
| `SHL` | alu | C ← bus shifted left 1, 0 into LSB | FS p.5 |
| `SHC` | alu | C ← bus rotated left 1 | FS p.5 |
| `Ld` | shift_counter | n ← bus<4..0> | FS p.5, Class 5 |
| `Decr` | shift_counter | n ← n − 1 | FS p.5, Class 5 |
| `CONin` | con_logic | CON ← cond(bus) | FS p.5, Class 5 |
| `End` | control_unit (internal) | step counter ← T0 | FS p.2 |
| `Goto6` | control_unit (internal) | step counter ← T6 | FS p.5, Class 5 |
| `Stop` | control_unit (internal) | Run ← 0 (opcode 31) | *new, not from course* |

ALU op signals are **one-hot**: at most one of `ADD..SHC` is active per step.
If none is active, the ALU result is `32'h0` (don't care; `Cin` should not be on).

### Bus sources (one-hot; exactly the `*out` signals)

| Signal | Value put on `bus` |
|--------|--------------------|
| `Rout` | `rf_out` (R[selected]) |
| `BAout` | `rf_out` (0 if R0 selected) |
| `PCout` | `PC` |
| `MDout` | `MD` |
| `Cout` | `C` |
| `C1out` | `c1_ext` |
| `C2out` | `c2_ext` |

At most **one** of these 7 may be active in any cycle. The testbench bus
contention checker asserts this on every cycle.

### Status signals (into `control_unit`)

| Signal | From | Meaning |
|--------|------|---------|
| `CON` | con_logic | result of the last `CONin` |
| `n_eq_0` | shift_counter | n = 0 |
| `Done` | memory | the memory access requested this step is complete |
| `IR[31:0]` | ir | full instruction (opcode, and IR<4..0> for shifts) |

---

## 4. Module ports

Direction is from the module's point of view. All widths in bits.

### 4.1 `regfile.v` — owner: Alon

| Port | Dir | Width | Description |
|------|-----|-------|-------------|
| `clk` | in | 1 | |
| `rst` | in | 1 | clears R0..R31 to 0 (see §6, Q3) |
| `ra`, `rb`, `rc` | in | 5 each | register fields from `ir` |
| `Gra`, `Grb`, `Grc` | in | 1 each | register select (at most one active) |
| `Rin` | in | 1 | R[selected] ← `bus_in` on posedge |
| `BAout` | in | 1 | forces `rf_out = 0` when selected register is R0 |
| `bus_in` | in | 32 | |
| `rf_out` | out | 32 | R[selected], or 0 when `BAout` and selected = R0 |

Notes: R0 is a normal register (it can be written and read with `Rout`).
`Rout` is not a port here; it only selects `rf_out` in `bus.v`.
If none of `Gra/Grb/Grc` is active, the selected register is R0.

### 4.2 `ir.v` — owner: Alon

| Port | Dir | Width | Description |
|------|-----|-------|-------------|
| `clk` | in | 1 | |
| `rst` | in | 1 | IR ← 0 (decodes as nop) |
| `IRin` | in | 1 | IR ← `bus_in` on posedge |
| `bus_in` | in | 32 | |
| `IR` | out | 32 | full register (to control_unit) |
| `op` | out | 5 | IR<31..27> |
| `ra`, `rb`, `rc` | out | 5 each | IR<26..22>, IR<21..17>, IR<16..12> |
| `cond` | out | 3 | IR<2..0> |
| `c1_ext` | out | 32 | sign-extended IR<21..0> |
| `c2_ext` | out | 32 | sign-extended IR<16..0> |

### 4.3 `alu.v` (ALU + A + C) — owner: Alon

| Port | Dir | Width | Description |
|------|-----|-------|-------------|
| `clk` | in | 1 | |
| `bus_in` | in | 32 | ALU input B and input to A |
| `Ain` | in | 1 | A ← `bus_in` on posedge |
| `Cin` | in | 1 | C ← ALU result on posedge |
| `ADD`, `SUB`, `AND`, `OR`, `NOT`, `CeqB`, `INC4`, `SHR`, `SHRA`, `SHL`, `SHC` | in | 1 each | one-hot op select |
| `C` | out | 32 | C register (to bus mux) |

A and C are not reset (always written before being read).

### 4.4 `shift_counter.v` — owner: Alon

| Port | Dir | Width | Description |
|------|-----|-------|-------------|
| `clk` | in | 1 | |
| `bus_in` | in | 5 | `bus[4:0]` |
| `Ld` | in | 1 | n ← `bus_in` on posedge |
| `Decr` | in | 1 | n ← n − 1 on posedge |
| `n_eq_0` | out | 1 | combinational: n = 0 |

`Ld` and `Decr` are never active together.

### 4.5 `con_logic.v` — owner: Alon

| Port | Dir | Width | Description |
|------|-----|-------|-------------|
| `clk` | in | 1 | |
| `bus_in` | in | 32 | value to test (R[rc]) |
| `cond` | in | 3 | IR<2..0> |
| `CONin` | in | 1 | CON ← cond(bus) on posedge |
| `CON` | out | 1 | registered result |

| `cond` | Meaning | CON |
|--------|---------|-----|
| 0 | never | 0 |
| 1 | always | 1 |
| 2 | zero | bus = 0 |
| 3 | nonzero | bus ≠ 0 |
| 4 | plus | bus<31> = 0 |
| 5 | minus | bus<31> = 1 |
| 6, 7 | unused | 0 |

### 4.6 `mem_if.v` (MA + MD) — owner: Eran

| Port | Dir | Width | Description |
|------|-----|-------|-------------|
| `clk` | in | 1 | |
| `bus_in` | in | 32 | |
| `MAin` | in | 1 | MA ← `bus_in` on posedge |
| `MDbus` | in | 1 | MD ← `bus_in` on posedge |
| `Read` | in | 1 | MD ← `mem_rdata` on posedge (this is MDrd; see §6, Q1) |
| `mem_rdata` | in | 32 | data from memory |
| `MA` | out | 32 | to memory address |
| `MD` | out | 32 | to bus mux and to memory write data |

`MDbus` and `Read` are never active together.

### 4.7 `memory.v` — owner: Eran

| Port | Dir | Width | Description |
|------|-----|-------|-------------|
| `clk` | in | 1 | |
| `addr` | in | 32 | from MA (byte address) |
| `wdata` | in | 32 | from MD |
| `Read` | in | 1 | `rdata` ← M[addr] |
| `Write` | in | 1 | M[addr] ← `wdata` on posedge |
| `rdata` | out | 32 | combinational read |
| `Done` | out | 1 | MVP: `Done = Read \| Write` (zero latency) |

- Size: **4 KB = 1024 words**, indexed by `addr[11:2]`. `addr[1:0]` and
  `addr[31:12]` are ignored (word-aligned accesses only).
- Big-endian: the byte at the lowest address is bits <31..24> of the word.
  Since all accesses are full words, this only matters for `asm.py` / `iss.py`.
- Loaded with `$readmemh` from a file given by parameter `MEMFILE`. One 32-bit
  word per line; line k is byte address 4k.

### 4.8 `bus.v` — owner: Alon

| Port | Dir | Width | Description |
|------|-----|-------|-------------|
| `Rout`, `BAout`, `PCout`, `MDout`, `Cout`, `C1out`, `C2out` | in | 1 each | source select |
| `rf_out` | in | 32 | from regfile (used by `Rout` and `BAout`) |
| `PC` | in | 32 | |
| `MD` | in | 32 | |
| `C` | in | 32 | |
| `c1_ext`, `c2_ext` | in | 32 each | from ir |
| `bus` | out | 32 | 0 if no source active |

### 4.9 `control_unit.v` — owner: Eran

| Port | Dir | Width | Description |
|------|-----|-------|-------------|
| `clk` | in | 1 | |
| `rst` | in | 1 | step ← T0, Run ← 1 |
| `IR` | in | 32 | |
| `CON` | in | 1 | |
| `n_eq_0` | in | 1 | |
| `Done` | in | 1 | |
| all §3 signals except `Wait`, `End`, `Goto6`, `Stop` | out | 1 each | |
| `End` | out | 1 | exposed for the step-count assertion |
| `step` | out | 4 | current step (0 = T0), for testbench and waves |
| `halt` | out | 1 | `~Run` |

Behavior:
- Step counter is 4 bits (T0..T15).
- `End` → step ← T0. `Goto6` → step ← T6. `Wait & ~Done` → step holds.
  Otherwise step ← step + 1.
- `Stop` → Run ← 0. When Run = 0, the step counter holds and **all** control
  outputs are 0.

### 4.10 `datapath.v` / `src_top.v` — owners: both

`datapath.v` holds the PC register and wires all modules above.

| PC | Behavior |
|----|----------|
| reset | PC ← 0 |
| `PCin` | PC ← `bus` on posedge |

`src_top.v` ports:

| Port | Dir | Width | Description |
|------|-----|-------|-------------|
| `clk` | in | 1 | |
| `rst` | in | 1 | |
| `halt` | out | 1 | processor executed `stop` |

The testbench reads internal state hierarchically (e.g. `dut.dp.rf.R[5]`,
`dut.mem.M[25]`); the exact hierarchy names are fixed when `src_top.v` is written.

---

## 5. Opcodes in scope

Phase A: nop 0, ld 1, st 3, la 5, br* 8, add 12, addi 13, sub 14, and 20,
or 22, not 24, stop 31.
Phase B: ldr 2, str 4, lar 6, brl* 9, neg 15, andi 21, ori 23,
shr 26, shra 27, shl 28, shc 29.

---

## 6. Open questions / changelog

| # | Question | Status |
|---|----------|--------|
| Q1 | FS p.4 shows both `MDrd` and `Read`. In this contract `Read` alone loads MD from memory (MDrd = Read). Check FS p.4 and the Class 5 fetch: is that how the course uses them? | open |
| Q2 | FS p.9 lists 31 among the unused opcodes, but Heuring & Jordan define `stop` = 31 and `nop` = 0. We use 31 / 0. Confirm. | open |
| Q3 | `rst` clears R0..R31. Not from the course; it avoids X values when comparing against `iss.py`. Keep? | open |

| Date | Change | By |
|------|--------|----|
| 2026-10-24 | First draft | Alon |
