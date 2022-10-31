# Plenum board design notes

## Components

### [PCI18F66J60](http://ww1.microchip.com/downloads/en/DeviceDoc/39762f.pdf) microcontroller
Microcontroller with I/Os: SPI, I2C, Ethernet, GPIO. *Would be good to write clean SPI driver, since many peripherals use SPI*.

Key interfaces: 


### [LTC2983](https://www.analog.com/media/en/technical-documentation/data-sheets/2983fc.pdf) temperature sensor readout
Use with RTDs (9 3-wire RTDs per chip possible, if sharing compensation pin). Total of 18 sensors read out by 2 copies of chip. **Find RTDs**. 

### [BMI270](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bmi270-ds000.pdf) IMU
Not strictly necessary, but may complement NSROC GNC data.

* Use 3V3 for input voltage. Max typical current draw is 970 µA @ VDD=1V8, in accelerometer and gyro performance mode, in ambient 25 ºC. This is 1.746 mW, or 0.529 µA equivalent current at 3V3 input.
* Wish to use in SPI mode, but I2C is default interface. To enable SPI, the CS pin needs to see a rising edge (start low, go high) after the BMI270 powers up. This must be done after every powerup or reset to enable SPI communication. Otherwise, can set the `spi_en` bit in the register `NV_CONF` to permanently set SPI as the data interface.
* Has internal temperature sensor, updates when gyro enabled.