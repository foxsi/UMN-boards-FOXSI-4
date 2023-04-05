# Plenum board design notes

## Components

### [PCI18F66J60](http://ww1.microchip.com/downloads/en/DeviceDoc/39762f.pdf) microcontroller
Microcontroller with I/Os: SPI, I2C, Ethernet, GPIO. *Would be good to write clean SPI driver, since many peripherals use SPI*.

#### Programming
Use the [PICkit 4 In-Circuit Debugger](https://ww1.microchip.com/downloads/en/DeviceDoc/50002751F.pdf). Has 8-pin connector to board (supply programmer with USB to laptop power, or power board separately). Connector is 0.1" sockets with pinout:


| Pin	| Description	|
|:-------|:------------|
| 1		| `nMCLR`		|
| 2		| `VDD`		|
| 3		| `GND` 		|
| 4		| `PGD`		|
| 5		| `PGC`		|
| 6 		| DNC		|
| 7		| Unused		|
| 8		| Unused		|

It would be nice to run this pinout verbatim down one side of the 2x8 power-to-plenum interboard stacking connector. 

The `PGC` and `PGD` pins are supposed to be kept as short as possible (datasheet p.46). 

Key interfaces: 

### SPI

### Ethernet

### GPIOs to reserve for debug

- UART interface
- An interrupt (INT3)
- Some analog inputs

### Sensing

#### [LTC2983](https://www.analog.com/media/en/technical-documentation/data-sheets/2983fc.pdf) temperature sensor readout
Use with RTDs (9 3-wire RTDs per chip possible, if sharing compensation pin). Total of 18 sensors read out by 2 copies of chip. **Find RTDs**. 

Interface with µC using SPI: `nCS` (for refdes `U19`) and `nCS2` (for refdes `US20`).

#### [BMI270](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bmi270-ds000.pdf) IMU
Not strictly necessary, but may complement NSROC GNC data.

* Use 3V3 for input voltage. Max typical current draw is 970 µA @ VDD=1V8, in accelerometer and gyro performance mode, in ambient 25 ºC. This is 1.746 mW, or 0.529 µA equivalent current at 3V3 input.
* Wish to use in SPI mode, but I2C is default interface. To enable SPI, the CS pin needs to see a rising edge (start low, go high) after the BMI270 powers up. This must be done after every powerup or reset to enable SPI communication. Otherwise, can set the `spi_en` bit in the register `NV_CONF` to permanently set SPI as the data interface.
* Has internal temperature sensor, updates when gyro enabled.

#### Hygrometry
From [Digikey](https://www.digikey.com/en/products/filter/humidity-moisture-sensors/529?s=N4IgjCBcoCwdIDGUBmBDANgZwKYBoQB7KAbRAGYBOSgBgFYAOEAmGgNjHeYrHIfIBMIALoEADgBcoIAMoSATgEsAdgHMQAXwJtKUUMkjps+IqRCtKVctzhgGlAOwjxUyLIUr1WkALpCEBka4BMSQZHZgdHQQBAJgAhxOsfFs9NxxvuQ06fF0DjE+uQx0Ob40ST7Flrqx5KmUbKUwvk3kME0wFRkwbI3JAp0l-Z1Mw47OIJLSckpqmgRtJQGomMGmYRR0vVvc5Fu9fZsHTKKTru6zXgRglPZ6SCvGIWbZwt4CAlT3gasmoeE0AAEWG4DGB3Ac4O0UJAdBhMBh5BhNAAdGBETQgQBbEHXGFgFFI3HgFFw4kCcGnKZuACqykUEgA8igALI4NBYACu8hwmg0GiAA), can't find a good panel-/chassis-mount sensor for this that we could stick in the focal plane. May need to do a mini board with the sensor mounted that goes inside the blanketing.

#### Pressure
Consider [MS5607](https://www.te.com/commerce/DocumentDelivery/DDEController?Action=srchrtrv&DocNm=MS5607-02BA03&DocType=Data+Sheet&DocLang=English) or [MS5803](https://www.te.com/commerce/DocumentDelivery/DDEController?Action=srchrtrv&DocNm=MS5803-01BA&DocType=Data+Sheet&DocLang=English) from TE. Both very small, inexpensive, with SPI interface.

#### Integrated gas sensor
Could also take temperature, pressure, humidity, and gas content measurement with [Bosch BME680](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bme680-ds001.pdf). But it is extremely tiny, at 9 mm^2 footprint area. 

[Github driver here](https://github.com/BoschSensortec/BME68x-Sensor-API).

#### Current sensing on Phil's board
Consider [AD7490](https://www.analog.com/media/en/technical-documentation/data-sheets/AD7490.pdf) 16:1 ADC to SPI converter. Can monitor 12x current sense resistors on Phil board with GND reference (put current sensing on low side), plus 4x voltages (5 V, 5.5 V, 12 V, 28 V in?). Need to normalize all voltages to 0-5 V range.

### Clock sync
Sequence of events:

1. `LAUNCH` signal (indicating launch about to occur, signal driver NSROC via power board) goes HIGH. 
2. Causes NOR-based SR-latch (refdes `U3` + `U4`) to latch HIGH (output signal `Q`. Nominally, `LAUNCH` remains HIGH until µC sets `UNLAUNCH` HIGH, nominally on receive `SHUTTER` signal HIGH (signal driver NSROC via power board). 
3. On next rising edge of `PPS` (signal driver NSROC via power board), `PPS` AND `LAUNCH`/`Q`, single-ended `SYNC` output of AND gate (refdes `U5`) goes HIGH, driving TTL inputs of differential transmitters (refdes `U6` - `U11`).
4. When µC receives `SHUTTER` HIGH, regardless of `LAUNCH` state, it will set `UNLAUNCH` output low after next observed `SYNC` falling edge. This will latch the SR into `nQ` state HIGH/`Q` state LOW, turning off the AND gate `U5` and choking `SYNC` throughput.

Alternative plan for step 4: µC simply notifies formatter of the time `SHUTTER` goes HIGH and continues sending `SYNC` to subsystems until power is cut. 

Note that µC must notify formatter of `SHUTTER` latch HIGH anyway so formatter can inform subsystems of imminent power off.

## Routing

- Note: Fusion360 model (as of Mar22 2023) has all connectors on BOTTOM side of board.
- MOVE ALL CONNECTORS TOP SIDE. 
- Keep all reflow ICs top side. 
- Hand-solder components can go wherever.