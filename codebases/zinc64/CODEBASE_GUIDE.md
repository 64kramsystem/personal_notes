# zinc64 Codebase Guide

- [zinc64 Codebase Guide](#zinc64-codebase-guide)
  - [Workspace layout](#workspace-layout)
  - [Where to start reading](#where-to-start-reading)
  - [Key architectural ideas](#key-architectural-ideas)
  - [Memory map essentials](#memory-map-essentials)
  - [Suggested reading order](#suggested-reading-order)
  - [Things to know upfront](#things-to-know-upfront)

## Workspace layout

| Crate           | Role                                                 |
| --------------- | ---------------------------------------------------- |
| `zinc64-core`   | All hardware components (CPU, VIC, SID, CIA, memory) |
| `zinc64-system` | Wires components together into a running C64         |
| `zinc64`        | Standalone app (OpenGL, audio, CLI)                  |
| `zinc64-loader` | File format loaders (PRG, CRT, TAP…)                 |
| `zinc64-debug`  | Remote debugger support                              |

---

## Where to start reading

1. **`zinc64-system/src/c64.rs`** — the system orchestrator. `build()` shows exactly how all chips are wired together, and `run_frame()` + `step_internal()` show the emulation loop.

2. **`zinc64-core/src/factory/`** — the `ChipFactory` and `Chip` traits. These are the glue. Every hardware component implements `Chip` (with `clock()`, `reset()`, `read()`, `write()`).

3. **`zinc64-core/src/cpu/`** — the 6510 CPU. `step()` drives the whole machine. The CPU calls a `TickFn` closure *once per clock cycle*, which clocks all other chips in lockstep.

4. **`zinc64-core/src/mem/`** — the PLA (memory mapper). Understand bank switching (5 bits → 32 memory modes) before reading any other component.

---

## Key architectural ideas

**Clock-driven synchronization via `TickFn`:** The CPU doesn't own other chips. Instead, it's given a closure at construction time:

```rust
let tick_fn = Rc::new(move || {
    vic.borrow_mut().clock();
    cia_1.borrow_mut().clock();
    cia_2.borrow_mut().clock();
    datassette.borrow_mut().clock();
    clock.tick();
});
```

This is called on every CPU cycle, keeping everything in sync cycle-accurately.

**I/O line abstractions:** Chips don't hold references to each other. They communicate through `Pin`, `IrqLine`, `IoPort` — thin shared wrappers (`Rc<Cell<T>>`). This is what makes the `no_std` port and component substitution feasible.

**Shared state pattern:** `Shared<T> = Rc<RefCell<T>>` everywhere for mutable chip state. Borrow panics at runtime rather than compile time — keep that in mind when tracing bugs.

---

## Memory map essentials

```
0xA000–0xBFFF   BASIC ROM  (switchable)
0xD000–0xDFFF   I/O space: VIC ($D000), SID ($D400), Color RAM ($D800), CIA1 ($DC00), CIA2 ($DD00)
0xE000–0xFFFF   Kernal ROM (switchable)
```

Bank selection is controlled by CPU I/O port `$0001` bits — see `zinc64-core/src/mem/pla.rs`.

---

## Suggested reading order

1. `zinc64-core/src/factory/` — traits (`Chip`, `Cpu`, `Mmu`, `Addressable`)
2. `zinc64-system/src/c64.rs` — wiring + loop
3. `zinc64-core/src/mem/pla.rs` — memory banking
4. `zinc64-core/src/cpu/cpu.rs` + `cpu_gen1/cpu6510.rs` — micro-operation CPU
5. `zinc64-core/src/video/vic.rs` — raster video (most complex component)
6. `zinc64-core/src/io/cia.rs` — timers, keyboard, joystick
7. `zinc64-core/src/sound/sid.rs` — delegates to `resid-rs` for synthesis

---

## Things to know upfront

- **PAL vs NTSC** config affects cycles/frame and clock frequency. The `SystemModel` enum carries all timing constants.
- The **VIC video pipeline** is the most intricate part: raster sequencer → gfx sequencer → sprite sequencer → mux unit → border unit.
- **Autostart** (`zinc64-system/src/autostart.rs`) hooks into the boot sequence by detecting when the CPU reaches `$A65C` (BASIC ready prompt).
- The CPU implements *all* undocumented 6502 opcodes, validated against Klaus2m5's functional test suite.
