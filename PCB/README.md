# PCB PikoAkustik v3

Revision of **PikoAkustik v2**. Focus: audio quality for audiometry, battery efficiency, and safety.

## Summary

- MCU: STM32F303 → **STM32L4**
- Audio: MAX98357A (Class-D, mono) → **ES8388** (stereo codec: DAC + headphone driver + ADC)
- 5V booster removed
- Relay used for safety and self-test, not for left/right selection
- Main power switch fixed (no leakage current when OFF)

## Changes by block

### Audio
- MAX98357A replaced with **ES8388**
  - Analog output (not PWM), lower noise
  - Stereo: left/right ear selected via I2S data, no relay needed
  - Built-in ADC for calibration self-test
- Audio data 16-bit → **24-bit**
- Added **47 µF coupling capacitors** on LOUT1/ROUT1
- Added **attenuator** (series R + shunt R) per channel: safer maximum level, more accurate low levels
- TRS jack: Tip = L, Ring = R, **Sleeve = GND**
- Removed: L1/L2, speaker filter, MAX98357A gain/SD_MODE setting

### Relay
- G5V-1 (5V coil) → **Omron G6K-2F-Y DC3** (DPDT, 3V coil)
- New function (fail-safe):
  - Relay OFF (default): output to **32 Ω dummy load** + ADC → headphone disconnected, anti-pop, self-test
  - Relay ON: output to headphone (during test)
- Coil powered from 3V3_STM, driven by 2N7002 with B5819W flyback diode

### Power
- **Audio Relay Booster removed** (MCP1640 / MT3608 and related parts)
- Main power switch:
  - Comparator Q3/Q5 (AO3401A) → **BCM857BS** (matched dual PNP)
  - R62/R63 10k → **100k**
  - SW10 moved **before** the ideal diode (VBAT) and **before** D10 (VDDUSB) → near-zero leakage when OFF
- 3 separate AP2112K-3.3 LDOs: STM32, ESP32, codec
- Codec rail: 0 Ω at input (can be replaced with ferrite bead), optional EN from GPIO (DNP)
- L3 (in parallel with LDO) removed
- Label fix: capacitors "4.7uH" → **4.7uF**

### MCU and connectivity
- STM32F303RB → **STM32L4** (SAI for I2S + MCLK, I2C to codec)
- ESP32: added **CH340C USB-UART** + auto-reset (2 NPN), via the charging USB port
- Reset/boot buttons kept (smaller size)
- microSD and SSD1306 OLED unchanged

## Notes

- Dummy load matches the headphone impedance in use (currently 32 Ω, JBL Tune 500)
- Attenuator values are calculated from headphone sensitivity and target maximum level
- Acoustic calibration per headphone is still required; the ADC self-test only checks the electrical path
