---
inclusion: always
---

# Product Overview

## Purpose

Provide a self-check diagnostic core for an obsolete STM32F407 battery-management-system board, so the board can be reused as a hardware-validation test bench instead of being discarded.

## Key features

- Capture raw RCC/FLASH registers and decode PLL clock configuration.
- Compute the clock tree under two HSE hypotheses (8 MHz and 25 MHz).
- Measure real HCLK with the DWT cycle counter, cross-checked against SysTick.
- Classify the result into a verdict with a ppm error and confidence flag.
- Expose all results through a plain C struct readable in a debugger Watch window.

## Target users

Embedded firmware engineers validating clock configuration on STM32F407 hardware, particularly where a wrong crystal / `HSE_VALUE` / `PLLM` combination produces a plausible but incorrect system clock.

## Success criteria

- The verdict is derived from measurement, not only from register reads.
- A measurement that matches none of the hypotheses is reported as `UNKNOWN`, never forced into a verdict.
- No reliance on a display, serial output, or host toolchain at runtime.
