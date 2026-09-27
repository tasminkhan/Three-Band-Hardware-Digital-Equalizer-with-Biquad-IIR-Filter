# Three-Band Digital Audio Equalizer — RISC-V (TinyQV) Peripheral

A three-band digital audio equalizer built as a standalone memory-mapped peripheral for the TinyQV RISC-V CPU, processing 16-bit audio through three parallel biquad IIR filters with independent bass, mid, and treble control.

![hdl](https://img.shields.io/badge/HDL-SystemVerilog%20%7C%20Verilog-blueviolet)
![sim](https://img.shields.io/badge/Sim-Icarus%20%7C%20cocotb-brightgreen)
![lang](https://img.shields.io/badge/Verification-Python%20%7C%20pytest-blue)
![dsp](https://img.shields.io/badge/Coeff_design-MATLAB-orange)
![target](https://img.shields.io/badge/target-RISC--V_TinyQV-red)

- [Project documentation](docs/info.md)

## What this is

[TinyQV](https://github.com/TinyTapeout/ttsky25a-tinyQV) is a RISC-V CPU for Tiny Tapeout. A peripheral extends the CPU by occupying a range in its memory map, so the CPU reads and writes it directly. This repository is the **standalone** peripheral: it talks over SPI through a custom Python class rather than through CPU instructions. For the full version driven by RISC-V commands, see [ttsky25a-tinyQVb_Digital-Equalizer](https://github.com/tasminkhan/ttsky25a-tinyQVb_Digital-Equalizer.git). The peripheral module is [here](src/peripheral.v).

## What it does

Three parallel biquad IIR filters split a 16-bit audio stream into three bands and recombine them into the equalized output, at a 44.1 kHz sampling rate:

- Low-pass (bass): 300 Hz cutoff
- Band-pass (mid): 300 Hz – 10 kHz
- High-pass (treble): 10 kHz cutoff

## Design

- Filter coefficients designed in MATLAB with standard Butterworth functions for the target cutoffs.
- Coefficients scaled by 2^14 to move from floating-point to fixed-point for hardware.
- Early prototyping in Vivado using SystemVerilog for fast iteration, producing the test [vectors](https://github.com/tasminkhan/Three-Band-Digital-Equalizer-with-Biquad-IIR-Filter/tree/2f834cafcb0823aef5aa384bd94849cc316f1970/vectors).

## Build and test

- Compiled with Icarus Verilog under a Makefile build for automated compile-and-simulate.
- Verified with cocotb: tests written in Python against a high-level interface, organized and run with pytest.

## Key design decisions

- Single-cycle latency: output is ready one clock after the input write, for real-time processing.
- 16-bit data path with 32-bit internal precision to prevent overflow during filter computation.
- Memory-mapped register interface for CPU integration and control.

## How to use

- Reset the peripheral to initialize all registers.
- Write an input sample to `0x08` (triggers processing).
- Read the output from `0x00` on the next clock cycle.
- Adjust gains by writing control bits to `0x04`.

## Plots

**Filter coefficients**

<p align="center"><img src="plots/Coefficients in Fixed Point Arithmatic.png" width="70%"></p>

**Filtered waveform**

<p align="center"><img src="plots/filtered_waves.png" width="80%"></p>

**Frequency analysis**

<p align="center"><img src="plots/frequency_analysis.png" width="80%"></p>
