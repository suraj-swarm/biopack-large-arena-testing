# BioPack STM32 Stimulation Firmware

Firmware for the BioPack STM32U375-based board. This project generates timed 40 Hz PWM stimulation signals for cerci and antenna actuation outputs.

## Hardware Target

* MCU: STM32U375
* Framework: STM32 HAL / STM32CubeIDE
* Timer used for stimulation: TIM2
* PWM frequency: 40 Hz
* PWM duty cycle: 50%

## Output Mapping

| Function      | MCU Pin | Timer Channel |
| ------------- | ------: | ------------: |
| Left Cerci    |     PA0 |      TIM2_CH1 |
| Left Antenna  |     PA1 |      TIM2_CH2 |
| Right Antenna |     PA2 |      TIM2_CH3 |
| Right Cerci   |     PA3 |      TIM2_CH4 |

## Stimulation Sequence

The firmware repeatedly performs the following sequence:

1. Stimulate both cerci for 500 ms.
2. Wait 5 seconds.
3. Stimulate the left antenna for 500 ms.
4. Wait 5 seconds.
5. Stimulate both cerci for 500 ms.
6. Wait 5 seconds.
7. Stimulate the right antenna for 500 ms.
8. Wait 5 seconds.
9. Repeat.

## PWM Configuration

TIM2 is configured as:

```c
htim2.Init.Prescaler = 149;
htim2.Init.Period = 1999;
sConfigOC.Pulse = 1000;
```

With a 12 MHz TIM2 clock:

```text
PWM frequency = 12,000,000 / ((149 + 1) * (1999 + 1)) = 40 Hz
```

A pulse value of `1000` gives approximately 50% duty cycle.

## Key Timing Defines

```c
#define STIM_DURATION_MS      500
#define CERCI_TO_ANTENNA_MS   5000
#define ANTENNA_TO_CERCI_MS   5000
```

## Build and Flash

1. Open the project in STM32CubeIDE.
2. Build the project.
3. Connect the board using ST-LINK.
4. Flash/debug the firmware.

## Notes

* The stimulation outputs are generated using hardware PWM, not software GPIO toggling.
* PA0–PA3 must remain configured as TIM2 alternate-function pins.
* Do not use `HAL_GPIO_WritePin()` on these pins while PWM is active.
* The project is configured for LDO supply mode using:

```c
HAL_PWREx_ConfigSupply(PWR_LDO_SUPPLY);
```

## Suggested Verification

Use an oscilloscope or logic analyzer to confirm:

* PA0 and PA3 output 40 Hz during cerci stimulation.
* PA1 outputs 40 Hz during left antenna stimulation.
* PA2 outputs 40 Hz during right antenna stimulation.
* Each stimulation burst lasts 500 ms.
