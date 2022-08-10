# Veloren

- [Veloren](#veloren)
  - [World generation](#world-generation)

## World generation

- `world` crate (author looks at `site2` mod)
- Sites are generated in two steps: at the beginning, then as as player steps into areas
- Only the high-level details are generated; low-level ones are filled in as needed
- Site type: `SiteKind` enum
  - Refactor is a proxy for the new code
- Site has a 2d `TileGrid` made of blocks
- On generation, the structures (e.g. road, house...) laying on a given tile are examined, and the `Structure` trait implementation invoked for each structure
- `Painter` uses Constructive Solid Geometry in order to produce structures
  - Uses `Primitive`s
