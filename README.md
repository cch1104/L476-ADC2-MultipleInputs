# STM32 Dual ADC Voltage Display on 16x2 LCD

## Project Overview

This project uses an STM32 NUCLEO-L476RG board to read two analog inputs with ADC1 and display their converted voltages on a 16x2 LCD.

## Hardware Used

* STM32 NUCLEO-L476RG
* 16x2 LCD (HD44780 compatible, 4-bit mode)
* Potentiometer or analog signal sources
* Breadboard and jumper wires

## Features

* Reads two ADC channels using scan mode
* Converts raw ADC values to millivolts
* Displays Channel 3 on LCD line 1
* Displays Channel 4 on LCD line 2
* Updates every 1 second

## ADC Formula

```c
Voltage_mV = ADC_Value * 3300 / 4095;
```

## ADC Configuration Summary

```c
hadc1.Instance = ADC1;
hadc1.Init.Resolution = ADC_RESOLUTION_12B;
hadc1.Init.ScanConvMode = ADC_SCAN_ENABLE;
hadc1.Init.EOCSelection = ADC_EOC_SEQ_CONV;
hadc1.Init.ContinuousConvMode = ENABLE;
hadc1.Init.NbrOfConversion = 2;
```

### Meaning

* **12-bit resolution**: values from 0 to 4095
* **Scan mode enabled**: converts multiple channels automatically
* **2 conversions**: Channel 3 then Channel 4
* **Continuous mode**: repeats conversion sequence continuously

## Channel Order

```c
Rank 1 = ADC_CHANNEL_3
Rank 2 = ADC_CHANNEL_4
```

Conversion loop:

1. Read Channel 3
2. Read Channel 4
3. Repeat

## Main Loop Example

```c
float mv1, mv2;
char buff[16];
lcd_Init();
while (1)
{
    HAL_ADC_Start(&hadc1);

    HAL_ADC_PollForConversion(&hadc1,100);
    adcResult1 = HAL_ADC_GetValue(&hadc1);
    mv1 = ((float)adcResult1) * 3300.0 / 4095.0;

    lcd_Clear();
    lcd_Goto(0,0);
    sprintf(buff, "%7.2f", mv1);
    lcd_Puts(buff);

    HAL_ADC_PollForConversion(&hadc1,100);
    adcResult2 = HAL_ADC_GetValue(&hadc1);
    mv2 = ((float)adcResult2) * 3300.0 / 4095.0;

    lcd_Goto(0,1);
    sprintf(buff, "%7.2f", mv2);
    lcd_Puts(buff);

    HAL_ADC_Stop(&hadc1);
    HAL_Delay(1000);
}
```

## LCD Output Example

```text
1234.56
2890.12
```

## Recommended Beginner Settings

For easier control:

```c
ContinuousConvMode = DISABLE;
ScanConvMode = ENABLE;
NbrOfConversion = 2;
```

Then start once, read two values, stop.

## If Float Display Does Not Work

Add linker flag in STM32CubeIDE:

```text
-u _printf_float
```

## Future Improvements

* Add mV unit text
* Use DMA mode
* Add averaging filter
* Add bar graph display
* Use multi-channel sensors