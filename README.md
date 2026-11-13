# STM32F401RE FreeRTOS Scheduler & Kernel Trace Labs

Hardware-verified FreeRTOS multitasking and kernel scheduling experiments on an **STM32F401RE (ARM Cortex-M4)** Nucleo-64 board, profiled at microsecond resolution using **SEGGER SystemView** over ST-Link SWD/RTT.

## Hardware & Toolchain
* **MCU:** STM32F401RE (ARM Cortex-M4 @ 84 MHz, 512 KB Flash, 96 KB SRAM)
* **Board:** STMicroelectronics NUCLEO-F401RE (On-board ST-LINK/V2-1)
* **RTOS & Profiler:** FreeRTOS Kernel + SEGGER SystemView V4.10b (RTT Trace)
* **IDE:** STM32CubeIDE

---

## Lab 01: Preemptive vs. Cooperative Scheduling & Kernel Timing Benchmarks

Measured empirical timing parameters on STM32F401RE (`84 MHz` system clock, `configTICK_RATE_HZ = 1000`):
* **SysTick ISR Execution Latency:** `5.00 µs – 5.54 µs` per `1 ms` tick interrupt.
* **Context Switch (`Scheduler` / `PendSV`) Latency:** `~6.6 µs` to save/restore task stack frames (`PSP`).

### 1. Preemptive Blocking & Idle Task Fallback (`001.SVdat` / `preemptive_blocking_trace.png`)
* **Config:** `configUSE_PREEMPTION = 1`
* **Behavior:** `SysTick` preempts `Task-2` after `1.013 ms` and switches to equal-priority `Task-1`. Both tasks execute briefly and invoke `vTaskDelay(1000)`, transitioning from **Running** to **Blocked (Delayed)** state. With zero user tasks in the Ready list, the scheduler dispatches the **Idle Task** (`System Idle`, priority `0`).

### 2. Preemptive Round-Robin Time-Slicing (`002.SVdat` / `round_robin_timeslicing_trace.png`)
* **Config:** `configUSE_PREEMPTION = 1`, `configUSE_TIME_SLICING = 1`
* **Behavior:** Two equal-priority tasks (`Task-1` and `Task-2`, both `Priority = 2`, `724 B` stack allocated) run continuously without blocking. Every `1 ms` `SysTick` interrupt (`5.512 µs` ISR + `6.6 µs` switch overhead) slices CPU time equally, giving each task **`~993.4 µs`** of net execution per tick.

### 3. Cooperative Scheduling with Explicit `taskYIELD()` (`003.SVdat` / `cooperative_yield_trace.png`)
* **Config:** `configUSE_PREEMPTION = 0`
* **Behavior:** `SysTick` interrupts still fire every `1 ms` (`5.00 µs – 5.21 µs`), but `ISR Exit` returns directly to the active task without invoking the scheduler. `Task-2` runs continuously for **`2.222 ms`** across multiple ticks until it explicitly calls `taskYIELD()`, handing control to `Task-1` (`2.210 ms`).

### 4. Cooperative Scheduling with `vTaskDelay()` Blocking (`004.SVdat` / `cooperative_blocking_trace.png`)
* **Config:** `configUSE_PREEMPTION = 0`
* **Behavior:** `Task-2` executes uninterrupted across multiple `SysTick` boundaries for **`2.280 ms`** before yielding to `Task-1`. `Task-1` runs briefly, calls `vTaskDelay()` to enter the **Blocked** state, and immediately returns CPU control back to `Task-2`.

---

## Repository Structure
```text
STM32F401RE-FreeRTOS-Scheduler-Trace-Labs/
├── README.md
└── 01_Preemptive_vs_Cooperative_Trace/
    ├── main.c
    ├── FreeRTOSConfig.h
    └── traces/
        ├── preemptive_blocking_trace.png
        ├── round_robin_timeslicing_trace.png
        ├── cooperative_yield_trace.png
        ├── cooperative_blocking_trace.png
        ├── 001.SVdat
        ├── 002.SVdat
        ├── 003.SVdat
        └── 004.SVdat
