# APB3 GPIO Controller with Secret Output Toggle

A SystemVerilog implementation of an 8-bit GPIO controller with APB3 interface, featuring a hidden secret pin toggle mechanism.

## Overview

This project implements a simple 8-bit GPIO controller with one APB register. The controller follows the standard APB3 protocol and includes a special hidden behavior that toggles a secret output pin when a specific write sequence is detected.

## Features

- **APB3 Interface**: Standard 2-cycle APB protocol support
- **8-bit GPIO Register**: Read/write access to GPIO data register
- **Secret Pin Toggle**: Hidden behavior triggered by a specific write sequence
- **Synthesizable RTL**: SystemVerilog implementation ready for synthesis

## Project Structure

```
apb_gpio_ctrl/
├── rtl/                          # RTL source files
│   └── apb_gpio_ctrl.sv         # Main module implementation
├── harness/                      # Test harness
│   ├── sources/                  # Reference implementations
│   └── tests/                    # Test files
│       ├── apb_gpio_ctrl_test_hidden.py  # Cocotb testbench
│       └── pytest_apb_gpio_ctrl.py        # Pytest runner
├── docs/                         # Documentation
│   └── Specification.md          # Detailed specification
└── pyproject.toml                # Python project configuration
```

## Module Interface

```systemverilog
module apb_gpio_with_secret_toggle (
    input  logic        pclk,        // APB clock
    input  logic        presetn,      // Active-low async reset
    input  logic        psel,         // APB select
    input  logic        penable,      // APB enable
    input  logic        pwrite,       // APB write/read
    input  logic [31:0] paddr,        // APB address
    input  logic [31:0] pwdata,       // APB write data
    output logic [31:0] prdata,       // APB read data
    output logic        pready,       // APB ready
    output logic        secret_pin    // Secret pin (toggles on magic sequence)
);
```

## Register Map

| Offset | Name | Access | Bits | Description | Reset |
|--------|------|--------|------|-------------|-------|
| 0x00   | GPIO | RW     | [7:0]| GPIO data register | 0x00 |

**Note**: Only the lower 8 bits of the 32-bit APB interface are used.

## Normal Functionality

- **Writes**: Write operations to the GPIO register instantly update the internal value
- **Reads**: Read operations return the current GPIO register value
- **APB Protocol**: Standard 2-cycle APB protocol - `pready` is asserted one cycle after `penable`

## Special Hidden Behavior

The `secret_pin` output toggles (0→1 or 1→0) when the following exact sequence occurs:

1. **Three consecutive writes** to offset 0x00
2. **Write values**: `0x02` → `0x04` → `0x08` (in this exact order)
3. **Timing**: Atleast 3 `pclk` cycles between each write
4. **No reads**: No read operations between the three writes

When the sequence is correctly detected, `secret_pin` flips its current value.

