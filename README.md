APB3 GPIO Controller with Secret Output Toggle

A SystemVerilog implementation of an 8-bit GPIO controller with an APB3 interface, featuring a hidden secret-pin toggle mechanism triggered by a defined write sequence.

Overview

This project implements a minimal 8-bit GPIO controller accessible through a standard APB3 bus.
In addition to normal read/write behavior, the controller contains a hidden state machine that toggles a secret_pin output when a strict 3-write sequence is detected with correct timing and no interruptions.

Features

APB3 Interface
Fully compliant 2-phase APB protocol (setup → enable)

8-bit GPIO Register
Read/write access to a single 8-bit register at address offset 0x00

Hidden Secret Toggle Mechanism
A special 3-write sequence causes secret_pin to toggle

Synthesizable SystemVerilog RTL
Clean synchronous-reset implementation, lint-friendly

Project Structure
apb_gpio_ctrl/
├── rtl/
│   └── apb_gpio_ctrl.sv                   # Golden RTL (Option B)
├── harness/
│   ├── sources/
│   └── tests/
│       ├── apb_gpio_ctrl_test_hidden.py   # Cocotb testbench
│       └── pytest_apb_gpio_ctrl.py        # Pytest wrapper
├── docs/
│   └── Specification.md                   # Full spec
└── pyproject.toml

Module Interface
module apb_gpio_with_secret_toggle (
    input  logic        pclk,       // APB clock
    input  logic        presetn,    // Active-low synchronous reset
    input  logic        psel,       // APB select
    input  logic        penable,    // APB enable phase
    input  logic        pwrite,     // 1 = write, 0 = read
    input  logic [31:0] paddr,      // APB address
    input  logic [31:0] pwdata,     // APB write data
    output logic [31:0] prdata,     // APB read data
    output logic        pready,     // Always ready (single-cycle response)
    output logic        secret_pin  // Hidden state machine output
);

Register Map
Offset	Name	Access	Bits	Description	Reset
0x00	GPIO	RW	[7:0]	GPIO data register	0x00

Only bits [7:0] are functional; upper bits read as zero.

Normal Behavior

Write
Writing to address 0x00 updates the 8-bit GPIO register

Read
Reading from 0x00 returns the current GPIO value

APB Protocol
The design follows the standard APB3 2-phase handshake:

Setup cycle: psel = 1, penable = 0

Enable cycle: psel = 1, penable = 1

Transfer sampled on rising edge during enable cycle

pready is always 1, indicating no wait states

Secret Hidden Behavior (Sequence Detection)

secret_pin toggles only if the following sequence occurs:

Three APB writes to the GPIO register at address 0x00

Data values must be exactly:

0x02 → 0x04 → 0x08


in this strict order

Timing constraint:
At least 3 full pclk cycles between each write strobe
(i.e., between the enable phases)

No Reads:
Any read between the writes aborts the sequence

Abort on wrong address:
Any write to a different address aborts the sequence

Abort on non-increasing or unexpected values

If the sequence completes correctly, secret_pin flips (0→1 or 1→0)
exactly one cycle after the third write strobe.
