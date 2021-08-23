# Tiled

## Map/Tilesets

There can be more tilesets for a map.

Tilesets can be detached (metadata saved in `.tsx` files), or embedded in the map!

### Tile properties

Tile properties can be editer in the tile editor (icon in the `Tileset` tab).

The `Type` property can be used by games (e.g. Macroquad has a special `jumpthrough` type).

## Layers

A map can have more layers, for example:

- background: eg. terrain
- foreground: above background, eg. base of a tree
- top: above all (incl. player), eg. top of a tree

There are different ways to design the z coordinate, but layers are very convenient.

### Collisions/Objects

Collisions can be designed via "object" layers (tre are different ways to design them, but this way is convenient).


