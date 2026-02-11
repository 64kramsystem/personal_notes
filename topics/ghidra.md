# Ghidra

- [Ghidra](#ghidra)
  - [Ide](#ide)
  - [Semantics](#semantics)
  - [Shortcuts/Bindings](#shortcutsbindings)
  - [RubyrDragon scripts](#rubyrdragon-scripts)
    - [Set label at arbitrary address](#set-label-at-arbitrary-address)


## Ide

Scale rendering: `support/launch.properties` → `VMARGS_LINUX=-Dsun.java2d.uiScale` → set to `2`
- `VMARGS=-Dfont.size.override=` scales the IDE fonts, but not the disassebly!

## Semantics

- `Data` (r. click)          → set the data type at an address  (e.g. byte)
- `Edit Function` (r. click) → set `unknown` calling convention
                                `Function return…` → set `Datatype` to `void`
- `Window` → `Memory map`   → add a block of memory mapping the chosen address, (set `uninitialized`)
- `Window` → `Symbol Table` → manage symbols

Labels can't be added mid-instruction via GUI. For that, see [Set label at arbitrary address](#set-label-at-arbitrary-address).

## Shortcuts/Bindings

- `D`                : disassemble selected instruction(s)
- `C`                : clear code bytes (convert to data)
- `L`                : label address
- `R`                : add reference
- `;`                : add comment
- `F`                : edit function
- `E`                : set equate (constant)
- `Ctrl` + `T`       : symbol table

- `Alt` + `←`/`→`       : navigate history
- `Ctrl` + `Shift` + `F` : show references
- `Ctrl` + `D`           : add bookmark
- `G`                    : go to (adress, etc.)

## RubyrDragon scripts

```rb
#@category Malware/DOS                      # Sets the folder
#@runtime Ruby                              # "Ruby" causes a warning (but works)
#@keybinding Ctrl-Shift-V
```

### Set label at arbitrary address

```rb
#@category Assembly

java_import 'ghidra.program.model.symbol.SourceType'

symbol_table = currentProgram.getSymbolTable
namespace    = currentProgram.getGlobalNamespace

address = toAddr("07c0:012A")

symbol_table.createLabel(address, "INT13_ORIGINAL_VECTOR_OFS", namespace, SourceType::USER_DEFINED)

puts "OK: labeled #{address}"
```
