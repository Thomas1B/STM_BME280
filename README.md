# STM BME280 using I2C

This library will work with BMP280, however no humidity measurement is available

Datasheet: [BME280](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bme280-ds002.pdf)

## BME280 Features

The BME280 sensor is a 3 in 1 sensor" temperature, pressure, and humidity.

## BME280 Setup
It is always recommended to read the datasheet for full understanding.


### In STM32CubeMX

Pick what I2C pins/channel you want to use.

### In STM32CubeIDE

Copy `BME280.C` to "Core/Src", and `BME280.h` to "Core/Inc"

In the header file BME280.h, change these to the I2C channel you using
```C
extern I2C_HandleTypeDef hi2c1;
#define bme_i2c (hi2c1)
```


In main.c:

```C
/* USER CODE BEGIN Includes */
#include <stdio.h>
#include "BME280.h"
/* USER CODE END Includes */
```

```C
/* USER CODE BEGIN PV */
BME280_Init_t bme280Init = {
		/* Configuration of BME280_Init_t structure with desired settings
		 * for oversampling, mode, standby time, filter and SPI interface */
		.OverSampling_T = OVERSAMPLING_1,
		.OverSampling_P = OVERSAMPLING_1,
		.OverSampling_H = OVERSAMPLING_1,
		.Mode = BME280_NORMAL_MODE,
		.T_StandBy = T_SB_1000,
		.Filter = FILTER_OFF,
		.SPI_EnOrDıs = SPI3_W_DISABLE
};

BME280_Raw_Data_t rawData; // Variable to hold raw data from the BME280 sensor
BME280_Data_t allData; // Variable to hold processed data from the BME280 sensor
/* USER CODE END PV */
```

```C
/* USER CODE BEGIN 2 */
BME280Init(bme280Init); // Initialize the BME280 sensor with the specified settings

for (uint8_t addr = 1; addr < 128; addr++) {
	if (HAL_I2C_IsDeviceReady(&hi2c1, addr << 1, 1, 100) == HAL_OK) {
		printf("Found device at 0x%02X\r\n", addr);
	}
}

HAL_Delay(1000);
/* USER CODE END 2 */
```

```C
/* USER CODE BEGIN WHILE */
while (1) {
	/* USER CODE END WHILE */

	/* USER CODE BEGIN 3 */

	rawData = BME280_RawData(); // Read raw data from the BME280 sensor

	printf("Using Separate Functions:\r\n");
	printf("Temperature: %.2f C\r\n", BME280_getTemperature(rawData.tempr));
	printf("Pressure: %.2f hPa\r\n", BME280_getPressure(rawData.pressr));
	printf("Humidity: %.2f %%\r\n", BME280_getHumidity(rawData.humr));
	printf("Altitude (P): %.2f m\r\n\n",
			BME280_getAltitude(rawData.pressr));

	allData = BME280_getAllData(); // Get all processed data in a structure
	printf("Using All in One Function:\r\n");
	printf("Temperature: %.2f C\r\n", allData.Temperature);
	printf("Pressure: %.2f hPa\r\n", allData.Pressure);
	printf("Humidity: %.2f %%\r\n", allData.Humidity);
	printf("Altitude (P): %.2f m\r\n\n", allData.Altitude);

	HAL_Delay(1000);
}
/* USER CODE END 3 */
```

```C
/* USER CODE BEGIN 4 */
int __io_putchar(int ch) {
//	HAL_UART_Transmit(&huart2, (uint8_t*) &ch, 1, HAL_MAX_DELAY); // Transmit the character over UART
	ITM_SendChar(ch); // Send the character to the SWO console (for debugging)
	return ch;
}
/* USER CODE END 4 */
```


## Handling Offsets