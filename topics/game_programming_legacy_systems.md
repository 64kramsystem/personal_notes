# Game Programming on Legacy Systems

- [Game Programming on Legacy Systems](#game-programming-on-legacy-systems)
  - [Video modes: 13h](#video-modes-13h)
  - [Input](#input)

## Video modes: 13h

Int `10h, AH=0(set mode), AL(mode)=13h`

- 320x200
- 8 bit
- uses a (6 bit -> 262k) palette
- A000:0000 -> A000:FBFF

Video card memories were very slow.

## Input

Getting input:

- joystick
  - must be calibrated, because the terminal position values increase with the CPU speed (!), as they're based on counting, not timing
  - neutral position is never 0
  - neutral (0,0) value is used to detect that a joystick is not connected
- keyboard
  - call int `16h, AH=01(check if ready)` to verify if the key is ready (`jnz`=ready), then int `16h, AH=00(get scan code)`
  - each key has a scan code, which is not the ASCII code
  - shift states (ctrl, shift...) are in a word bitset at location `0417h`
- mouse: on windows, goes through drivers

