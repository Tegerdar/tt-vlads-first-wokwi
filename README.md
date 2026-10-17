![](../../workflows/gds/badge.svg) ![](../../workflows/docs/badge.svg) ![](../../workflows/fpga/badge.svg)

# Vlad's first Wokwi design — hex 7-segment decoder + 4-bit counter

A [Tiny Tapeout](https://tinytapeout.com) project built from individual logic gates in [Wokwi](https://wokwi.com/projects/478036993243342849).

It drives a 7-segment display with a hexadecimal digit (`0`–`9`, `A`–`F`). The digit comes either from four input switches or from a built-in 4-bit up-counter, selected by a mode input.

- **Author:** Vladislavs Lazdāns
- **Language:** Wokwi (gate-level schematic)
- **Size:** 1×1 tile
- **Simulation:** <https://wokwi.com/projects/478036993243342849>
- **Datasheet:** [docs/info.md](docs/info.md)

## Features

- **Hex decoder:** a 4-bit value `X3..X0` is decoded to segments `a`–`g`. Each segment is its own sum-of-products made from AND/OR/NOT gates.
- **4-bit up-counter:** four D flip-flops count up by one on each rising edge of `clk`, wrapping from `F` to `0`.
- **Mode select:** `ui[4]` drives four 2:1 multiplexers. `0` shows the switch value and `1` shows the counter value.
- **Decimal point:** `uo[7]` lights when the counter reaches `F` (all four bits are 1).
- **Reset:** pulling `rst_n` low clears the counter to `0`.

## Block diagram

```
 ui[3:0] (X3..X0) ──┐
                    ├─► 4× MUX 2:1 ──► hex → 7-seg decoder ──► uo[6:0] (a..g)
 counter Q3..Q0 ────┘        ▲
                             │
 ui[4] (MODE) ───────────────┘

 clk ──► 4-bit counter (D-FFs + XOR/AND carry chain) ──► Q3..Q0
 rst_n ─► NOT ─► FF reset              Q3·Q2·Q1·Q0 ──► uo[7] (DP)
```

### Counter logic

| Bit | Next state (`D`) | Carry out |
|-----|------------------|-----------|
| Q0  | `¬Q0`            | —         |
| Q1  | `Q1 ⊕ Q0`        | `C2 = Q1·Q0` |
| Q2  | `Q2 ⊕ C2`        | `C3 = Q2·C2` |
| Q3  | `Q3 ⊕ C3`        | —         |

`DP = Q3·C3`. This is 1 only when the counter is at `1111` (F).

### Segment equations

`x3..x0` are the multiplexer outputs and `'` means NOT.

| Segment | Output | Equation |
|---------|--------|----------|
| a | `uo[0]` | `x2'x0' + x3'x1 + x2x1 + x3x0' + x3'x2x0 + x3x2'x1'` |
| b | `uo[1]` | `x2'x0' + x2'x1' + x3'x1'x0' + x3'x1x0 + x3x1'x0` |
| c | `uo[2]` | `x3'x2 + x3x2' + x1'x0 + x3'x0 + x3'x1'` |
| d | `uo[3]` | `x3x1' + x3'x2'x0' + x2'x1x0 + x2x1'x0 + x2x1x0'` |
| e | `uo[4]` | `x2'x0' + x1x0' + x3x2 + x3x1` |
| f | `uo[5]` | `x1'x0' + x2x0' + x3x2' + x3x1 + x3'x2x1'` |
| g | `uo[6]` | `x2'x1 + x1x0' + x3x2' + x3x0 + x3'x2x1'` |

The segment outputs are active-high, so they suit a common-cathode display such as the 7-segment display on the Tiny Tapeout demo board.

## Pinout

| Pin | Direction | Function |
|-----|-----------|----------|
| `ui[0]`–`ui[3]` | in | `X0`–`X3`: hex digit from switches (`X0` = LSB) |
| `ui[4]` | in | `MODE`: 0 = switches, 1 = counter |
| `ui[5]`–`ui[7]` | in | unused |
| `uo[0]`–`uo[6]` | out | segments `a`–`g` |
| `uo[7]` | out | `DP`: lit when the counter is at F |
| `uio[7:0]` | — | unused |
| `clk` | in | counter clock |
| `rst_n` | in | active-low counter reset |

## How to test

### In Wokwi

1. Open the [simulation](https://wokwi.com/projects/478036993243342849) and press ▶.
2. **Switch mode** (DIP 5 / `ui[4]` off): set DIP 1–4 to a binary value. The display shows that value as a hex digit.
3. **Counter mode** (DIP 5 on): set the clock slide switch to manual and pulse the clock with the push button, or switch it to the clock generator. Each pulse adds one to the displayed digit. The decimal point lights at `F`, and the next pulse wraps back to `0`.
4. Press the reset button (`rst_n`) to set the counter back to `0`.

### On the Tiny Tapeout demo board

Use the on-board DIP switches for `ui[4:0]` and the on-board 7-segment display for `uo[7:0]`. Clock the counter manually (single-step) or with a slow clock of a few Hz so you can watch it count.

## Repository layout

| Path | Contents |
|------|----------|
| `info.yaml` | Project metadata and pinout. `wokwi_id` points to the Wokwi project. |
| `docs/info.md` | Datasheet text used by the Tiny Tapeout docs build |
| `src/` | Template support files (`cells.v`, `config.json`). The netlist is fetched from Wokwi by CI. |
| `test/` | cocotb test scaffolding from the template (no tests written yet) |
| `.github/workflows/` | `gds`, `docs`, `fpga` CI workflows |

## Build

The GitHub Actions workflows fetch the gate netlist from Wokwi using `wokwi_id`, harden it to GDS, and build the docs. To change the design, edit it in Wokwi, then push any change to this repo (for example, to `info.yaml` or the docs) to trigger a rebuild.

## About Tiny Tapeout

Tiny Tapeout is an educational project that makes it cheaper and easier to get digital and analog designs manufactured on a real chip. See <https://tinytapeout.com>, the [FAQ](https://tinytapeout.com/faq/), and the [digital design lessons](https://tinytapeout.com/digital_design/).

## License

See [LICENSE](LICENSE).
