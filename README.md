# AMBA APB (Advanced Peripheral Bus) Protocol Implementation & Verification

An RTL design and functional verification suite for ARM's **AMBA APB (Advanced Peripheral Bus)** protocol. This repository implements an APB Master (Bridge/Requester), an APB Slave (Completer/Memory Interface), and a comprehensive testbench architecture demonstrating synchronous read/write bus operations, state machine transitions, and wait-state handling.

---

## 📌 Table of Contents
- [Overview](#overview)
- [Protocol Specifications & Signals](#protocol-specifications--signals)
- [Architecture & FSM Operation](#architecture--fsm-operation)
- [Read/Write Timing Diagrams](#readwrite-timing-diagrams)
- [Repository Structure](#repository-structure)
- [Simulation & Verification](#simulation--verification)
- [How to Run](#how-to-run)
- [Key Features](#key-features)
- [References](#references)

---

## 📖 Overview

The **Advanced Peripheral Bus (APB)** is part of the ARM AMBA multi-bus hierarchy, optimized for minimal power consumption and reduced interface complexity. Unlike high-throughput pipelined buses (e.g., AXI, AHB), APB is non-pipelined and unbursted, making it the industry standard for interfacing with low-bandwidth control peripherals such as:
- UART / SPI / I2C controllers
- General Purpose I/O (GPIO)
- Timers & Watchdog modules
- Configuration registers and interrupt controllers

---

## 🔌 Protocol Specifications & Signals

| Signal | Direction (Master $\rightarrow$ Slave) | Description |
| :--- | :---: | :--- |
| `PCLK` | System $\rightarrow$ Both | APB bus clock. All transfers are timed on rising edge. |
| `PRESETn` / `rst_n` | System $\rightarrow$ Both | Active-LOW asynchronous/synchronous system reset. |
| `PADDR[31:0]` | Master $\rightarrow$ Slave | APB address bus. Drives peripheral register address. |
| `PSELx` | Master $\rightarrow$ Slave | Peripheral select line. Asserted high when slave is addressed. |
| `PENABLE` | Master $\rightarrow$ Slave | Transfer enable. Indicates access phase of transfer. |
| `PWRITE` | Master $\rightarrow$ Slave | Transfer direction: `HIGH` = Write, `LOW` = Read. |
| `PWDATA[31:0]` | Master $\rightarrow$ Slave | Write data bus driven during write cycles (`PWRITE = 1`). |
| `PRDATA[31:0]` | Slave $\rightarrow$ Master | Read data bus driven by slave during read cycles (`PWRITE = 0`). |
| `PREADY` | Slave $\rightarrow$ Master | Handshake ready flag. Allows slave to insert wait states when LOW. |
| `PSLVERR` | Slave $\rightarrow$ Master | *(Optional/APB3+)* Indicates a transfer failure/error condition. |

---

## ⚙️ Architecture & FSM Operation

The APB transfer cycle operates strictly according to a 3-state Finite State Machine (FSM):