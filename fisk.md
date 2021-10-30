# Fisk

- [Fisk](#fisk)
  - [Classes](#classes)
  - [Temp registers](#temp-registers)

## Classes

- `Fisk`
  - `#[u]imm`: generates an `[U]Imm<size>` operand based on the value
- `Imm<size>`: Subclass of ImmediateOperand

## Temp registers

- `Fisk::Registers::Temp`
- `@register`: set on assignment
- `@start_point`, `@end_point` (set on release)

Related methods

- `Fisk@temp_registers`
- `Fisk#release_register`, `Fisk#release_all_registers`
- `Fisk#assign_registers(list)`: assign registers from the list to the `@temp_registers`, computing their activity based on start/end point
