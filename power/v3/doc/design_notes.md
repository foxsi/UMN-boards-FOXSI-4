# Power

## Status:

- Inheriting from Phil May 31 2023
- Need to put schematic in KiCad, do layout, and order.

## To do:

- Make all the packages in KiCad
    - [LT8645S](datasheets/LT8645S.pdf)
        - connect `NC`s to `GND`
    - [LT8612](datasheets/LT8612.pdf)
        - connect `NC`s to `GND`
    - [AD780](datasheets/AD780.pdf)
        - do not connect `DNC`s. Do not connect `O/P_select`.
    - [ACS724](datasheets/ACS724.pdf)
    - [G3VM-31WR](datasheets/G3VM-31WR.pdf)
    - [AD7490](datasheets/AD7490.pdf)
        - connect `NC`s to `AGND`
    - [MAX7317](datasheets/MAX7317.pdf)
    - [Gecko](datasheets/G125-MHX0805M4-2AD2ADP.pdf)

## Board layout    

### Mechanical
- Grow outline by 0.1"/0.05" on a side:
    - 4.7 x 2.45 in
    - Holes stay same place

    
### Placement
- Setup DRC and netclasses.
- Make 1 nuclear switch/sense group layout (e.g. for `CMOS2`) and replicate to the rest. Measure before replicating to make sure they all fit.
    - include pours, vias, traces, testpoints.
- Place and route the regulators.
- **NOTE**: AC5724 current sense chips want a 0.05" wide SLOT underneath to isolate current pass side from sensing side. Instead, just add a plane break beneath.
    - Use multilayer rule area, keepout copper
    - Don't need crazy sensitivity, so probably not a big deal


### Stackup
1. **TOP**: traces, local pours
2. **IN1**: reference planes for TOP. Separate clean/digital return planes.
3. **IN2**: power planes. Separate clean/digital/switched power.
4. **BOT**: traces, local pours. Include +28V_return heat sink area near switchers.

### Planes
There are many. Make sure to add:

* [ ] pour bottom layer, no soldermask, near heat sink 2-56 holes.

### Edits
* [x] remap LED driver for FORMATTER to bottom side 680 Ω
* [x] change MAX7317 pullups to 10kΩ or something
* [ ] star Earth GND in closer to Gecko
* [x] Add via in pad to all switching converters
* [x] And add large unsoldermasked area on bottom side