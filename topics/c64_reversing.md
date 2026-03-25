# C64 Reversing

- [C64 Reversing](#c64-reversing)
  - [Addressing modes](#addressing-modes)
  - [Conventions](#conventions)
  - [Memory locations](#memory-locations)
  - [Kernal routines](#kernal-routines)
    - [SCNKEY ($FF9F)](#scnkey-ff9f)
    - [GETIN ($FFE4)](#getin-ffe4)

## Addressing modes

- `STA ($nn,X)`: indexed indirect : pointer at (nn + X)

## Conventions

- Routines returning a single byte, store it in `A`

## Memory locations

- 24    : cursor column shadow register; holds the current horizontal cursor position (0-39) and is updated by the Kernal's character output routines.
- D6    : cursor row shadow register (0-24)
- D018  : VIC-II memory control register. Bits 7-4 select the screen RAM location within the current VIC bank (as a multiple of $0400). Bits 3-1 select the character set base address.
- DD00  : CIA2 Port A. Bits 0-1 (inverted) select the VIC-II memory bank out of four possible \$4000-sized regions. %00 = bank 3 (\$C000-\$FFFF).

## Kernal routines

### SCNKEY ($FF9F)

- Scans the keyboard matrix via CIA1
- Updates the internal key buffer at $0277-$0280 (10-byte circular buffer)
- Sets $C6 (number of keys in buffer)
- Normally called by the IRQ handler 60 times/second - if a game disables IRQs, it calls this manually to keep the keyboard working

### GETIN ($FFE4)

- Reads one character from the current input device (default: keyboard buffer)
- Returns the PETSCII code in A, or $00 if no key is pending
- Doesn't block; returns 0 if nothing is there
