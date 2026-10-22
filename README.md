# Quadrature encoder decoder

![gds](https://github.com/Joonsense/tt-order-quadrature/actions/workflows/gds.yaml/badge.svg) ![test](https://github.com/Joonsense/tt-order-quadrature/actions/workflows/test.yaml/badge.svg) ![docs](https://github.com/Joonsense/tt-order-quadrature/actions/workflows/docs.yaml/badge.svg)

Tiny Tapeout project `tt_um_joonsense_quadrature`, designed with the [chip-lab order flow](https://joonsense.github.io/chip-order/): a written spec and a cycle-level Python model went in, this repository came out.

## Spec

Reads a rotary encoder (channels A and B) and keeps an 8 bit position count.

- Inputs `ui_in[0]` = A, `ui_in[1]` = B are asynchronous, so each passes through a two flop synchronizer before use.
- The decoder compares the synchronized state with the state it saw one clock earlier. With state = (B << 1) | A, the forward
  Gray sequence is 0 -> 1 -> 3 -> 2 -> 0. One step forward adds 1 to the count, one step backward subtracts 1 (8 bit wrap).
  No change does nothing. A jump of two steps (both channels changed at once) is illegal: the count is unchanged and the
  sticky error flag is set.
- `uio_in[0]` = CLEAR: while high at a rising edge, count and error flag are cleared (this takes priority over counting).
- `uo_out` = count. `uio_out[7]` = error flag, the only `uio` output (`uio_oe` = 0x80).
- Reset clears the synchronizers, the previous state, count and error flag.

## Pins

| Pin | Signal |
|---|---|
| `ui[0]` | A |
| `ui[1]` | B |
| `uo[0]` | COUNT0 |
| `uo[1]` | COUNT1 |
| `uo[2]` | COUNT2 |
| `uo[3]` | COUNT3 |
| `uo[4]` | COUNT4 |
| `uo[5]` | COUNT5 |
| `uo[6]` | COUNT6 |
| `uo[7]` | COUNT7 |
| `uio[0]` | CLEAR |
| `uio[7]` | ERROR |

## Verification

| Check | Result |
|---|---|
| RTL vs model (`test/model.py`) | 11,997 cycles bit exact, 3 seeds, mid run reset |
| Yosys netlist vs model | 11,997 cycles bit exact |
| Mutation test | 8/8 injected faults caught |
| sky130 pre-layout | 683 um2, 74 cells, predicted 1x1 |

Placement, routing, DRC, LVS and the gate level test run in this repository's GitHub Actions (badges above).

## Test it

```bash
cd test
pip install -r requirements.txt
make            # RTL against the Python model
```

Not fabricated yet. Tiny Tapeout tiles are bought per shuttle at app.tinytapeout.com.
