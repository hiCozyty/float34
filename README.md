# Float30

A 35-key column-staggered split keyboard with wireless support, built on ZMK.

![Float30](https://img.shields.io/badge/status-in%20development-yellow)

## Features

- **35 keys** (18 left + 17 right) in a 5-column column-staggered layout
- **Wireless BLE** split using Nice!Nano v2 controllers
- **Reversible design** — PCB and case work for both hands
- **ZMK firmware** with 5 configurable layers and mouse key support
- **USB-C** per half (via Nice!Nano)

## Hardware

- 2× Nice!Nano v2
- 35× Kailh Choc v1 (PG1350) switches
- 35× Kailh hotswap sockets
- 1× 750mAh LiPo battery per half (recommended)
- (Optional) PMW3610 optical trackball on right half

### Pinout

| Function | Left GPIO | Right GPIO |
|----------|-----------|------------|
| Row 0    | P0.20     | P0.20      |
| Row 1    | P0.08     | P0.08      |
| Row 2    | P0.04     | P0.04      |
| Row 3    | P0.05     | P0.05      |
| Col 0    | P0.17     | P0.17      |
| Col 1    | P0.13     | P0.13      |
| Col 2    | P0.24     | P0.24      |
| Col 3    | P1.15     | P1.15      |
| Col 4    | P0.06     | P0.06      |
| Col 5    | P0.07     | P0.07      |

Diode direction: row-to-column.

## Firmware

This repo builds ZMK firmware for both halves automatically via GitHub Actions.

### Pre-built firmware

Download the latest `.uf2` files from the [Actions tab](https://github.com/hiCozyty/float30/actions):

- `float30_left.uf2` — left (peripheral, BLE)
- `float30_right.uf2` — right (central, USB)

### Flash

1. Double-tap the reset button on the Nice!Nano to enter bootloader mode
2. Copy the `.uf2` file onto the mass storage device that appears
3. The board will reboot automatically

### Build locally

```sh
west init -l config/
west update
west zephyr-export
west build -s zmk/app -b nice_nano_v2 -- -DSHIELD=float30_left -DZMK_CONFIG="${PWD}/config"
cp build/zephyr/zmk.uf2 float30_left.uf2
west build -s zmk/app -b nice_nano_v2 -- -DSHIELD=float30_right -DZMK_CONFIG="${PWD}/config"
cp build/zephyr/zmk.uf2 float30_right.uf2
```

## Keymap

| Layer | Name | Function |
|-------|------|----------|
| 0     | Default | QWERTY alphas |
| 1     | Nav     | Navigation, arrows, numbers |
| 2     | Sym     | Symbols, brackets |
| 3     | Fn      | F1-F12, numpad |
| 4     | Scroll  | Mouse scroll (placeholder) |

Activate layers 1-3 by holding the `mo 1` thumb key on the right half.

## License

MIT
