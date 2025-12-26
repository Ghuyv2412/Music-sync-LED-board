# Music-sync-LED-board

A real-time audio spectrum visualizer implemented on the **STM32F407VET6**, synchronizing an **8x8 WS2812 LED matrix** with ambient music captured via a **MAX4466** microphone.

The system performs high-speed Fast Fourier Transform (FFT) to convert time-domain audio signals into frequency bins, visualized across 8 distinct frequency bands with smoothed transitions.

---

### Authors

* Ghuyv2412 
* trhieuuit
* huynhtoaiii
---

### Hardware Components

* **MCU:** STM32F407VET6 (Cortex-M4 with FPU)
* **Sensor:** MAX4466 Electret Microphone Amplifier
* **Display:** 8x8 WS2812B RGB LED Matrix
* **Debugger:** ST-Link V2
* **Communication:** UART CP2102 (for telemetry and debugging)

---

### Folders

**firmware_stm32**: core implementation using CMSIS-DSP and HAL drivers.

* **audio_processing**: implementation of FFT logic, Hamming windowing, and DC offset removal.
* **led_drivers**: WS2812 timing control via PWM+DMA to ensure flicker-free animation.
* **dsp_utils**: frequency band splitting, log-scaling, and smoothing algorithms.

**hardware_design**: wiring schematics and pinout configuration.

* **pinout**: mapping for ADC1 (PA0), TIM3, and PE4 (K0 button).

---

### Technical Workflow

1.  **Audio Sampling**: Analog signals are captured via **ADC1 (PA0)**. **DMA** is used for continuous buffer collection triggered by **TIM3** to ensure a stable sampling rate.
2.  **Pre-processing**: 
    * **DC Offset Removal**: Calculates the average and subtracts it from the signal.
    * **Normalization**: Scales amplitude to ±1.0 range.
    * **Windowing**: Applies a **Hamming window** to minimize spectral leakage.
3.  **FFT Execution**: Utilizes the **CMSIS-DSP** library (`arm_rfft_fast_f32`) for real-time FFT, followed by `arm_cmplx_mag_f32` to calculate the magnitude spectrum.
4.  **Frequency Analysis**: The spectrum is divided into **8 frequency bands**. The peak value in each region is selected and adjusted using a `band_gain[]` array to enhance high-frequency sensitivity.
5.  **Smoothing & Mapping**: 
    * **Smoothing**: Weighted averaging between new and previous frames reduces flicker.
    * **Log-scaling**: Maps amplitudes to an 8-level height scale for the LED matrix.
6.  **User Interaction**: An external interrupt on **PE4 (K0 button)** allows mode switching with a 200ms software debounce.

---

### Notes

* The FPU (Floating Point Unit) on the STM32F407 must be enabled to handle `f32` DSP operations efficiently.
* The WS2812 update frequency is synchronized with the FFT processing loop via `ws2812_demos_tick`.
* Ensure the MAX4466 VCC is stable to minimize noise in the ADC readings.
