# Ghidra

- [Ghidra](#ghidra)
  - [IDE](#ide)
    - [IDE operations](#ide-operations)
    - [Shortcuts/Bindings](#shortcutsbindings)
  - [Labels](#labels)
  - [Data types](#data-types)
  - [Functions](#functions)
  - [Flow override (CALL)](#flow-override-call)
  - [Useful operations](#useful-operations)
    - [Operations on memory blocks](#operations-on-memory-blocks)
      - [Map an extra file to a memory block](#map-an-extra-file-to-a-memory-block)
    - [Create references to statically unknown locations](#create-references-to-statically-unknown-locations)
  - [Scripts (Ruby Dragon)](#scripts-ruby-dragon)
    - [Set label at arbitrary address](#set-label-at-arbitrary-address)
    - [Remap references in a range, to an equivalent segment](#remap-references-in-a-range-to-an-equivalent-segment)
    - [Create a data structure](#create-a-data-structure)


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

If Ghidra doesn't know what segment is in DS (or a similar segment register), it can't resolve memory references to the correct memory block.

Fix for direct memory references (e.g., `MOV AX, [0x8002]`):

→ Right-click on the instruction → Set Register Values → Set the value

Fix for immediate loads (e.g., `MOV SI, 0x8002`):

Setting the register value has no effect — Ghidra creates the reference from the immediate value alone, ignoring DS.

→ Right-click on the instruction → References → Add/Edit References → manually add a reference

## Scripts (Ruby Dragon)

WATCH OUT! Don't add `@runtime Ruby`, or the script will fail with `Script is not supported`.

```rb
# @category Malware/DOS          # Script folder
# @keybinding Ctrl-Shift-V       # Optional

# Both versions work
currentProgram
$current_program

# toAddr() is in the global scope
$current_api.toAddr
```

Ignore warnings about redefined constants.

### Set label at arbitrary address

```rb
java_import 'ghidra.program.model.symbol.SourceType'

symbol_table = currentProgram.getSymbolTable
namespace    = currentProgram.getGlobalNamespace

address = toAddr("07c0:012A")

symbol_table.createLabel(address, "INT13_ORIGINAL_VECTOR_OFS", namespace, SourceType::USER_DEFINED)

puts "OK: labeled #{address}"
```

### Remap references in a range, to an equivalent segment

```rb
NEW_SEGMENT = "07c0"

tx_id = currentProgram.start_transaction("Remap 0000:8xxx references")
success = false

begin
  ref_manager = currentProgram.reference_manager
  start_addr  = toAddr("0000:8000")
  end_addr    = toAddr("0000:81ff")

  addr = start_addr
  while addr <= end_addr
    refs = ref_manager.get_references_to(addr).to_a   # snapshot to avoid iterator mutation
    unless refs.empty?
      # Compute the equivalent address in the virus segment (07c0)
      target_addr = toAddr("#{NEW_SEGMENT}:#{"%04x" % addr.segment_offset}")
      refs.each do |ref|
        # Add new reference pointing to the correct segment
        ref_manager.add_memory_reference(
          ref.from_address,
          target_addr,
          ref.reference_type,   # preserve original ref type (read/write/data)
          SourceType::USER_DEFINED,
          ref.operand_index     # preserve which operand the ref belongs to
        )
        ref_manager.delete(ref)
      end
      puts "Remapped #{addr} -> #{target_addr} (#{refs.size} ref#{refs.size == 1 ? '' : 's'})"
    end
    addr = addr.next   # advance one byte at a time
  end

  success = true
ensure
  # Rollback on exception
  currentProgram.end_transaction(tx_id, success)
end
```

### Create a data structure

```rb
java_import 'ghidra.program.model.data.StructureDataType'
java_import 'ghidra.program.model.data.ByteDataType'
java_import 'ghidra.program.model.data.WordDataType'
java_import 'ghidra.program.model.data.ArrayDataType'
java_import 'ghidra.program.model.data.CharDataType'
java_import 'ghidra.program.model.data.DataTypeConflictHandler'

tx_id = currentProgram.startTransaction("Create BPB structure")
success = false

begin
  dtm    = currentProgram.dataTypeManager
  struct = StructureDataType.new("BPB", 0)

  struct.add(ArrayDataType.new(CharDataType::dataType, 8, 1), "oem_name",           "OEM identifier")
  struct.add(WordDataType::dataType,                          "bytes_per_sector",    "")
  struct.add(ByteDataType::dataType,                          "sectors_per_cluster", "")
  struct.add(WordDataType::dataType,                          "reserved_sectors",    "")
  struct.add(ByteDataType::dataType,                          "fat_count",           "")
  struct.add(WordDataType::dataType,                          "root_entries",        "")
  struct.add(WordDataType::dataType,                          "total_sectors",       "")
  struct.add(ByteDataType::dataType,                          "media_descriptor",    "")
  struct.add(WordDataType::dataType,                          "sectors_per_fat",     "")
  struct.add(WordDataType::dataType,                          "sectors_per_track",   "")
  struct.add(WordDataType::dataType,                          "heads",               "")
  struct.add(WordDataType::dataType,                          "hidden_sectors",      "")

  # Register the type
  dt = dtm.addDataType(struct, nil)

  # Clean existing associations, and associate
  START_ADDR = toAddr("07c0:0003")
  currentProgram.listing.clearCodeUnits(START_ADDR, START_ADDR.add(struct.length - 1), false)
  currentProgram.listing.createData(START_ADDR, dt)

  puts "Created data structure"

  success = true
ensure
  currentProgram.endTransaction(tx_id, success)
end
```
