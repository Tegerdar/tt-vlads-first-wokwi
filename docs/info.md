<!---

This file is used to generate your project datasheet. Please fill in the information below and delete any unused
sections.

You can also include images in this folder and reference them in the markdown. Each image must be less than
512 kb in size, and the combined size of all images must be less than 1 MB.
-->

## How it works

The design is a gate-level hexadecimal 7-segment decoder with a switchable 4-bit up-counter. It is built entirely from logic gates in Wokwi.

- **Input select:** four 2:1 multiplexers choose the 4-bit value to display. When `MODE` (`ui[4]`) is 0, the value comes from `X3..X0` (`ui[3:0]`). When `MODE` is 1, it comes from the counter.
- **Decoder:** each segment `a`–`g` (`uo[6:0]`) is a separate sum-of-products made from AND, OR and NOT gates. The decoder shows `0`–`9` and `A`–`F`. The outputs are active-high (for a common-cathode display).
- **Counter:** four D flip-flops count up on each rising edge of `clk`. `Q0` toggles every clock. Each higher bit is XORed with the carry from the bits below it (`C2 = Q1·Q0`, `C3 = Q2·C2`). The counter wraps from `F` to `0`.
- **Decimal point:** `uo[7]` is `Q3·Q2·Q1·Q0`, so it lights when the counter is at `F`.
- **Reset:** `rst_n` is inverted and drives the reset of all four flip-flops. Holding `rst_n` low clears the counter to `0`.

The counter keeps running in switch mode as well. `MODE` only selects which value is displayed.

## How to test

1. Reset the design (pull `rst_n` low, then release it).
2. **Switch mode:** set `ui[4]` = 0 and apply each value `0000`–`1111` on `ui[3:0]`. Check that the display shows `0`–`9`, `A`–`F`.
3. **Counter mode:** set `ui[4]` = 1 and single-step `clk`, or run it at a few Hz. The digit should go up by one per clock, `uo[7]` (DP) should light at `F`, and the next clock should wrap back to `0`.
4. Pull `rst_n` low at any point. The counter should return to `0`.

You can also try the design in the Wokwi simulation: <https://wokwi.com/projects/478036993243342849>

## External hardware

None beyond the Tiny Tapeout demo board. It uses the on-board DIP switches (`ui[4:0]`), the 7-segment display (`uo[7:0]`), and the clock/reset controls.
