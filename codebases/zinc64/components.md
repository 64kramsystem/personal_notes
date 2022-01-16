# Components

- [Components](#components)
  - [Notes](#notes)
  - [Descriptions](#descriptions)
    - [Clock](#clock)
    - [IoPort](#ioport)
    - [Pin](#pin)
    - [IrqLine](#irqline)

## Notes

A (`?`) in the header is something that it's not clear or certain.

## Descriptions

### Clock

Essentially, a simple counter.

### IoPort

Contains two data values (input and output) and a direction flag.

Supports an observer (function), which is invoked on any write (including the direction).

### Pin

Represents a hardware pin, with two end states (high/low) plus two intermediate ones (rising/falling).

### IrqLine

Has an 8-bit value ("signal"), which can be set on a per-bit basis. Is "low" when nonzero.
