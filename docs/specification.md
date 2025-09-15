Here is the specification document as requested.

-----

# Specification for SystemVerilog Testbench Environment

## 1\. Overview

This document specifies the architecture and components of a SystemVerilog testbench (`top_tb`) for the verification of a Device Under Test (`dut_top`). The environment is designed to be configurable and built using standard SystemVerilog constructs, with a build system managed by CMake and Verilator.

The primary goal is to create a modular and reusable testbench where bus interfaces can be configured to different AMBA protocols. This specification is intended to be parsed by an AI to generate the corresponding SystemVerilog code and build scripts.

## 2\. File Structure

All SystemVerilog modules and interfaces shall be implemented in their own files. The naming convention is as follows:

  - `top_tb.sv`
  - `reset_gen.sv`
  - `clk_gen.sv`
  - `dut_top.sv`
  - `memory_target_if.sv`
  - `dut_initiator_if0.sv`
  - `dut_initiator_if1.sv`
  - (and any necessary package/interface files)

A `CMakeLists.txt` file shall be provided at the root level to build the entire design using Verilator 5.x.

## 3\. Top-Level Architecture (`top_tb`)

The top-level module, `top_tb`, instantiates all necessary components for the test environment.

### 3.1. Hierarchy

The module hierarchy within `top_tb.sv` is as follows:

```systemverilog
module top_tb;

    // Global Parameters & Configurations
    // ...

    // Clock and Reset Generation
    clk_gen clk_gen_i ();
    reset_gen reset_gen_i ();

    // DUT Wrapper
    dut_top dut_top_i (
        // Connections to interfaces
    );

    // Bus Functional Models / Transactors
    memory_target_if memory_target_if_i ();
    dut_initiator_if0 dut_initiator_if0_i ();
    dut_initiator_if1 dut_initiator_if1_i ();

    // Interface Instantiations
    // ...

endmodule
```

## 4\. Module Specifications

### 4.1. `clk_gen()`

  - **File:** `clk_gen.sv`
  - **Description:** Generates a clock signal for the test environment.
  - **Ports:**
      - `output logic clk`: The generated clock signal.

### 4.2. `reset_gen()`

  - **File:** `reset_gen.sv`
  - **Description:** Generates a reset signal for the test environment.
  - **Ports:**
      - `output logic rst_n`: The generated active-low reset signal.

### 4.3. `dut_top()`

  - **File:** `dut_top.sv`
  - **Description:** A wrapper for the actual DUT. This module connects the DUT's bus interfaces to the testbench's abstract interface constructs. It also utilizes the **Verilog-DPI (Direct Programming Interface)** to expose specified internal DUT signals to the `top_tb` level for monitoring or debugging purposes. The specific signals to be exposed via DPI will be defined separately.

### 4.4. `memory_target_if()`

  - **File:** `memory_target_if.sv`
  - **Description:** A configurable memory model that acts as a target for one of the DUT's initiator ports. It connects to `dut_top` via a SystemVerilog `interface`.
  - **Protocol Configuration:** The AMBA protocol for this interface is globally selectable from the following options.
      - **`AXI4_128` (Default):** AXI4 protocol with a 128-bit data width and 48-bit address width.
      - **`AHB_32`:** AHB protocol with a 32-bit data width and 32-bit address width.
      - **`CHI_256`:** CHI protocol with a 256-bit data width and 48-bit address width.

### 4.5. `dut_initiator_if0()` & `dut_initiator_if1()`

  - **Files:** `dut_initiator_if0.sv`, `dut_initiator_if1.sv`
  - **Description:** Bus Functional Models (BFMs) that act as initiators to the DUT's target ports. They connect to `dut_top` via a SystemVerilog `interface`.
  - **Protocol Configuration:** The AMBA protocol for these interfaces is globally selectable from the following options.
      - **`AXI4_128` (Default):** AXI4 protocol with a 128-bit data width and 48-bit address width.
      - **`AXI4_64`:** AXI4 protocol with a 64-bit data width and 48-bit address width.
      - **`APB_64`:** APB protocol with a 64-bit data width and 32-bit address width.
      - **`NotPresent`:** The interface is not active. The module instance should not drive any signals or interact with the DUT.

## 5\. Interface Abstraction and Configuration

Bus protocols are abstracted using SystemVerilog `interface` and `package` constructs. A global configuration mechanism, likely using parameters or defines in `top_tb`, will specify which protocol is active for each interface.

### 5.1. Global Configuration

The protocol for each interface shall be specified globally within the `top_tb` module. If a configuration is not explicitly provided, the default value must be used.

**Example:**

```systemverilog
// In top_tb.sv
parameter MEM_TARGET_PROTOCOL = "AXI4_128";
parameter INITIATOR0_PROTOCOL = "AXI4_64";
parameter INITIATOR1_PROTOCOL = "NotPresent";
```

### 5.2. Handling "NotPresent"

When an interface's protocol is configured as `NotPresent`, its corresponding module (`dut_initiator_if0` or `dut_initiator_if1`) must be instantiated but must not affect the simulation. This means:

  - All its outputs should be tied to a high-impedance state (`'z`) or a benign logic level.
  - It should not drive the interface it is connected to.
  - Internal processes or tasks within the module should be disabled.

## 6\. Coding Guidelines

The following rules must be strictly adhered to during implementation.

1.  **Vendor Agnostic:** The code must **not** use any vendor-specific primitives, libraries, or attributes. All code shall conform strictly to the SystemVerilog (IEEE 1800-2017) standard to ensure portability.
2.  **Isolation of "NotPresent" Modules:** When an interface is configured as `NotPresent`, the corresponding module must be completely isolated and have no functional impact on the DUT or the testbench.
3.  **Scoped Use of Abstract Constructs:** The use of SystemVerilog constructs such as `interface`, `package`, and `modport` is to be **exclusively** for the abstraction of the AMBA bus interfaces between the testbench components and the `dut_top`. They should not be used for other general-purpose connections.

## 7\. Build System

A `CMakeLists.txt` file must be created to support building and simulating the entire testbench environment with **Verilator version 5.x**.

### 7.1. `CMakeLists.txt` Requirements

  - It should locate the Verilator executable.
  - It must compile all the `.sv` files in the correct order of dependency.
  - It should generate a single executable binary for the simulation.
  - It should allow passing top-level parameters (e.g., for protocol selection) to the Verilator command line.