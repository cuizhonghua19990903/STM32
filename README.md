# BMS Clock Integrity Diagnostic

**Category:** Resurrection

A self-check diagnostic core that brings an obsolete STM32F407 battery-management-system (BMS) board back into useful service as a hardware-validation test bench.

## The problem

On the legacy board, the crystal marked `Y1`, the `HSE_VALUE` compiler macro, and the PLL `PLLM` divider can disagree. Because `SystemCoreClock` and `HAL_RCC_GetSysClockFreq()` simply trust `HSE_VALUE`, software can report a correct clock while the silicon runs at a different one. Two common misconfigurations look identical in code but behave very differently in hardware:

- `HSE = 8 MHz` with `PLLM = 25` → VCO input 320 kHz (below the 1–2 MHz valid range), SYSCLK = 53.76 MHz
- `HSE = 25 MHz` with `PLLM = 25` → VCO input 1 MHz (valid), SYSCLK = 168 MHz

A wrong clock silently corrupts UART baud rates, timing windows, and protection task periods — the exact class of fault that is easy to miss with unit tests alone.

## What it does

`clk_verify` answers one question with evidence: **what is the real HCLK, and which configuration does it match?**

1. **Register capture** — snapshots `RCC->CR`, `RCC->PLLCFGR`, `RCC->CFGR` and `FLASH->ACR`, then decodes HSE/PLL state, `PLLM/N/P/Q`, AHB/APB dividers, and FLASH latency.
2. **Dual-hypothesis theory** — computes the full clock tree twice, once for `HSE = 8 MHz` and once for `HSE = 25 MHz`, so a wrong `HSE_VALUE` macro cannot hide the mismatch.
3. **Measured HCLK** — uses the DWT cycle counter together with SysTick over a user-controlled timing window to measure the actual HCLK, then cross-checks it with the SysTick reload value.
4. **Automatic verdict** — classifies the measurement against three hypotheses (HSI 16 MHz / HSE8 53.76 MHz / HSE25 168 MHz), reports the error in ppm, and marks confidence. A measurement more than 20% off is reported as `UNKNOWN` rather than guessed.

Results are written into a global report struct and can be inspected directly in a debugger Watch window — no display or serial output required on the target board.

### Using the measurement window

In the debugger Watch window:

1. Write `g_clk_win_run = 1` to start the window (default `g_clk_win_ms = 10000`).
2. Let the board run for the full window.
3. Write `g_clk_win_run = 0` to close it and produce the verdict in `g_clk.verdict` / `g_clk.verdict_text`.

Note: stopping the core freezes DWT and SysTick together. Pause the window with the 1 → run → 0 sequence; do not rely on wall-clock time while halted.

## Repository layout

```
.
├── Drivers/app/clk_verify.c        # diagnostic implementation
├── Drivers/app/clk_verify.h        # public report struct and verdict codes
├── .kiro/
│   ├── steering/                   # persistent project conventions
│   ├── hooks/                      # automated verification triggers
│   └── specs/                      # spec-driven development artifacts
├── LICENSE                         # MIT
└── README.md
```

## Built with Kiro

Development followed Kiro's spec-driven workflow: a structured spec defined the requirements, design, and task sequence before implementation; steering files carried the Keil MDK / STM32F407 conventions across sessions; and hooks automated verification after file changes. See `.kiro/` for the actual configuration.

## License

MIT — see [LICENSE](LICENSE).
