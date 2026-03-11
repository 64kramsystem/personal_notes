# Ghidra

- [Ghidra](#ghidra)
  - [IDE](#ide)
    - [IDE operations](#ide-operations)
    - [Shortcuts/Bindings](#shortcutsbindings)
  - [Labels](#labels)
  - [Data types](#data-types)
  - [Functions](#functions)
  - [Flow override (CALL)](#flow-override-call)
  - [References workflow](#references-workflow)
  - [Useful operations](#useful-operations)
    - [Operations on memory blocks](#operations-on-memory-blocks)
      - [Map an extra file to a memory block](#map-an-extra-file-to-a-memory-block)
    - [Create references to statically unknown locations](#create-references-to-statically-unknown-locations)
  - [Scripts (Ruby Dragon)](#scripts-ruby-dragon)
    - [Args/Nested script invocation](#argsnested-script-invocation)

## IDE

Scale rendering: `support/launch.properties` → `VMARGS_LINUX=-Dsun.java2d.uiScale` → set to `2`
- `VMARGS=-Dfont.size.override=` scales the IDE fonts, but not the disassebly!

### IDE operations

- RC → `Data`                              → set the data type at an address  (e.g. byte)
- `Window` → `Memory map`                  → add a block of memory mapping the chosen address, (set `uninitialized`)
- `Window` → `Symbol Table`                → manage symbols
- RC → `References` → `Add reference from` → set address and confirm

### Shortcuts/Bindings

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

## Labels

Labels can't be added mid-instruction via GUI. For that, see [Set label at arbitrary address](#set-label-at-arbitrary-address).

## Data types

Manage (eg. edit) via `Data Type Manager`

- `Structure`

## Functions

ASM style:

- Enable `Use Custom Storage`
- Set `Calling Convention` to `Unknown`
- Set return datatype to `void`
- Set params, e.g. `1`, `ushort`, `buffer segment`, `ES:2`

## Flow override (CALL)

Ghidra may assess, both legitimately and not, that a call doesn't return and place a `Flow Override: CALL_RETURN (CALL_TERMINATOR)`; when this is not desirable, remove the override for proper analysis:

Right-click → `Modify Instruction Flow` → set `-DEFAULT-`.

## References workflow

To manually generate (missing) references:

- Create the required memory block(s)
- Manually set the segment register assumption for the involved code region
  - Ghidra will now be able to compute the target address, but won't necessarily create the reference automatically
- Manually add the reference (`memory` type):
  - Ghidra will prefill the target address from the segment assumption
- Label the target address (if not already named)

**WATCH OUT**: If (auto-)analysis doesn't update refs after the fix, do `Clear code bytes` (`C`) then `Disassemble` (`D`) on the affected instruction(s).

## Useful operations

### Operations on memory blocks

Many operations can be done on memory blocks; in order to shrink one, split it:

- Open `Window` → `Memory Map`
- Click `Split Block`
- Delete the now unnecessary block

#### Map an extra file to a memory block

- Add the file to the program (**not** import)
- Check the memory block, and if not appropriate, delete and recreate

### Create references to statically unknown locations

## Scripts (Ruby Dragon)

WATCH OUT! Don't add `@runtime Ruby`, or the script will fail with `Script is not supported`.

```rb
#@category Malware/DOS          # Script folder
#@keybinding Ctrl-Shift-V       # Optional

# Both versions work
currentProgram
$current_program

# toAddr() is in the global scope
$current_api.toAddr
```

Ignore warnings about redefined constants.

### Args/Nested script invocation

Scripts can invoke each other:

```rb
runScript("ApplyDataType.rb", [NAME, "07c0:0003"].to_java(:string))
```

In order for a script to use default params, a workaround is required:

```rb
# `getScriptArgs` is available only from a runScript context.
args = begin; getScriptArgs; rescue NameError; []; end

from_imm_start = (args[0] || "8000").to_i(16)
```

