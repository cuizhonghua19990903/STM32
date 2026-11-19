# Requirements — Clock Integrity Diagnostic

## Introduction

Detect a mismatch between the physical HSE crystal, the `HSE_VALUE` macro, and the PLL `PLLM` divider on an STM32F407 BMS board, using a measurement-derived verdict rather than register assumptions.

## Requirements

### R1 — Register capture

The module SHALL capture `RCC->CR`, `RCC->PLLCFGR`, `RCC->CFGR` and `FLASH->ACR` into the report struct during initialization.

#### Scenario

When `clk_verify_init()` is called, the raw register values and decoded fields (HSE/PLL state, `PLLM/N/P/Q`, AHB/APB dividers, FLASH latency) are available in `g_clk`.

### R2 — Dual-hypothesis calculation

The module SHALL compute the full clock tree for both `HSE = 8 MHz` and `HSE = 25 MHz`.

#### Scenario

Given the captured PLL dividers, the report contains `hclk_hse8_hz` and `hclk_hse25_hz` plus the corresponding VCO input, VCO, SYSCLK, PCLK and USB frequencies.

### R3 — Measured HCLK

The module SHALL measure real HCLK using the DWT cycle counter over a user-controlled timing window and cross-check it with SysTick.

#### Scenario

When `g_clk_win_run` transitions 1 → 0 after the configured window, `meas_hclk_hz` is computed from the accumulated cycles and window length, and `meas_hclk_from_tick_hz` is computed independently from SysTick.

### R4 — Verdict classification

The module SHALL classify the measured HCLK into one of: HSI 16 MHz, HSE8 53.76 MHz, HSE25 168 MHz, or UNKNOWN.

#### Scenario

When the measured value is within tolerance of a hypothesis, `verdict` is set to that code with a ppm error and a human-readable `verdict_text`; when no hypothesis matches within 20%, `verdict` is `CLK_VERDICT_UNKNOWN`.

### R5 — No runtime host dependency

The module SHALL NOT require serial output or a display in its default configuration.

#### Scenario

With `CLK_VERIFY_PRINT = 0`, the verdict is fully observable through the debug Watch window.

### R6 — Watch-window controls

The module SHALL expose `g_clk_win_run`, `g_clk_win_ms` and `g_clk_meas_seq` as volatile variables.

#### Scenario

A debugger can start a measurement by writing 1 and close it by writing 0; `g_clk_meas_seq` increments once per completed measurement.
