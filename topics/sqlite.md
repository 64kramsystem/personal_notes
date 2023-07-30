# SQLite

- [SQLite](#sqlite)
  - [Syntax](#syntax)
  - [Client](#client)
    - [Display](#display)

## Syntax

```sql
CASE WHEN watched = 't' THEN 1 ELSE 0 END
IIF(watched = 't', 1, 0)                    # v3.32+
```

## Client

### Display

```sql
# Display in line mode, like MySQL `\G`
#
.mode line
```
