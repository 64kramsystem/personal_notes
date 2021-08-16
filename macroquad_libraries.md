# Macroquad Libraries

- [Macroquad Libraries](#macroquad-libraries)
  - [Tiled (`tiled`)](#tiled-tiled)
  - [Physics (`physic-platformer`)](#physics-physic-platformer)
  - [Entities management](#entities-management)
    - [Storage](#storage)
    - [Scene/Node graph](#scenenode-graph)
    - [Entity-Component](#entity-component)
  - [State machine](#state-machine)

## Tiled (`tiled`)

Types:

- `Map`     : layers, tilesets, raw map; doesn't include "raw" properties (e.g. width/height)
- `Layer`   : objects, w/h, tiles
- `Tile`    : id, tileset, attrs
- `Tileset` : texture, w/h, ...
- `Object`  : something that is in the map, but not a tile, e.g. a spawn point

WATCH OUT! There is an internal `Map`, whose fields can reached via `Map#raw_tiled_map`; it includes properties like width/height.

```rs
let tiled_map: tiled::Map = tiled::load_map(
    &tiled_map_json,
    &[("tileset.png", tileset), ("decorations1.png", decorations)], // textures
    &[],                                                            // external tilesets
).unwrap();

// Get tile at x/y (tile units, not pixels)
let tile: Option<&Tile> = tiled_map.get_tile(LAYER_NAME, x, y)

// Get tiles inside a rectangle
let tiles: TilesIterator = tiled_map.tiles(LAYER_NAME, rect);

// Get an object world coordinates; objects are inside layers.
let objects = tiled_map.layers["logic"].objects;
let macroquad_tiled::Object { world_x, world_y, .. } = objects[0];

// Draw the map
tiled_map.draw_tiles("main layer", Rect::new(0.0, 0.0, map_w, map_h), None);
```

## Physics (`physic-platformer`)

Types:

- `World`            : layers, (solids+colliders) + (actors+colliders)
- `StaticTiledLayer` : tiles
- `Collider`
- `Actor`            : only id
- `Solid`            : only id
- `Tile`             : tyle type (enum)

```rs
const JUMP_SPEED: f32 = -700.0;
const GRAVITY: f32 = 2000.0;
const MOVE_SPEED: f32 = 300.0;

// The actor position is owned/managed by the world (see move_*() calls in the game loop).
struct Player {
    collider: Actor,
    speed: Vec2,
}

struct Resources {
    whale: Texture2D,
    physics: World,
}

impl Resources {
    fn new() {
        // Map Tiled tile types to internal ones.
        let tile_types = tiled_map
            .tiles("main layer", None)
            .map(|(_, _, tile)| match tile {
                None => crate::Tile::Empty,
                Some(tile) if tile.attrs.contains("jumpthrough") => crate::Tile::JumpThrough,
                _ => crate::Tile::Solid,
            })
            .collect();

        // Create the world, and add the tile colliders.
        let mut world = World::new();
        world.add_static_tiled_layer(
            tile_types,                         // static colliders
            32., 32.,                           // tile w/h
            tiled_map.raw_tiled_map.width as _, // width
            1,                                  // tag
        );
    }
}

impl Player {
    fn new() {
      // When a new actor is added, create a collider for it, and add it to the world.
      // Use the collsion box dimensions here (which can be different from the sprite ones)
      let collider: Actor = world.add_actor(
          vec2(200., 100.), // pos
          36, 66,           // w/h
      );
    }

    fn update() {
        // Get the player position; the Actor is just an id wrapper.
        let player_pos: Vec2 = world.actor_pos(player.collider);
        let on_ground: bool = world.collide_check(player.collider, player_pos + vec2(0., 1.));

        if !on_ground {
            player.speed.y += GRAVITY * get_frame_time();
        }

        if is_key_down(KeyCode::Right) {
            player.speed.x = MOVE_SPEED;
        } ... // handle right
        } else {
            player.speed.x = 0.;
        }

        if is_key_pressed(KeyCode::Space) {
            if on_ground {
                player.speed.y = JUMP_SPEED;
            }
        }

        world.move_h(player.collider, player.speed.x * get_frame_time());
        world.move_v(player.collider, player.speed.y * get_frame_time());
    }
}
```

## Entities management

### Storage

```rs
storage::store(resources);

// Get a mutable reference to something in the storage
let mut resources: impl DerefMut<Target = Resources> = collections::storage::get_mut::<Resources>();
let world = &mut world.physics;
```

### Scene/Node graph

WATCH OUT! When finding nodes, no more than a single reference can be borrowed at a time (in any scope); subsequent finds will return None.

```rs
// Add an entity to the scene graph
scene::add_node(player);

impl Node for Player {
    fn update(mut node: RefMut<Self>) {
        // Find nodes
        let players: impl Iterator<Item = RefMut<Player>> = scene::find_nodes_by_type::<crate::nodes::Player>();

        for mut player in players {
            if condition {
              // Delete a node from the graph
              player.delete();
            }
        }

        // Find one node
        let node: Option<RefMut<dyn Camera>> scene::find_node_by_type::<Camera>();
    }

    fn draw(node: RefMut<Self>) {
        // ... draw texture
    }
}
```

### Entity-Component

Implemented mainly via `Node#provides()`/`scene::find_nodes_with::<T>()`; it's essentially an EC-based implementation of traits. Pure Rust traits won't work, because `Sized` types are required.

```rs
// Pseudo-trait definition, with the return types of its methods.
// Lens is used to return variable properties of the entity.
type Sproingable = (HandleUntyped, Lens<PhysicsBody>, Vec2);

impl scene::Node for Grenades {
    fn setup(mut grenade: RefMut<Self>) {
      // Pseudo-trait implementation
      // A hypothetical pure-trait implementation would fail here, because the types involved must be
      // Sized, so a trait object can't be returned.
      grenade.provides::<Sproingable>((
          grenade.handle().untyped(),
          grenade.handle().lens(|grenade| &mut grenade.body),
          GRENADE_SIZE,
      ));
    }
}

impl scene::Node for Sproinger {
    fn fixed_update(mut node: RefMut<Self>) {
      /* other logic here */

      for (_handle: HandleUntyped, mut body_lens: Lens<PhysicsBody>, size: Vec2) in scene::find_nodes_with::<Sproingable>() {
          if let Some(body) = body_lens.get() {
              if body.speed.length() > Self::STOPPED_THRESHOLD {
                  let body_hitbox = Rect::new(body.pos.x, body.pos.y, size.x, size.y);
                  if sproinger_rect.intersect(body_hitbox).is_some() {
                      body.speed.y = -Self::FORCE;
                      sproinger.has_sproinged = true;
                      Sproinger::animate(sproinger.handle());
                  }
              }
          }
      }
    }
}
```

## State machine

The state machine API is completely undocumented; these are the basic notions:

```rs
let mut state_machine: <RefMut<Player>> = StateMachine::new();

// Can add a state only if the SM is not in use.
// The state constants must be usize.
state_machine.add_state(Self::ST_NORMAL, State::new().update(Self::update_normal));
state_machine.add_state(Self::ST_DEATH, State::new().coroutine(Self::death_coroutine));
state_machine.add_state(Self::ST_INCAPACITATED, State::new().update(Self::update_incapacitated).coroutine(Self::incapacitated_coroutine));
state_machine.add_state(Self::ST_SHOOT, State::new().update(Self::update_shoot).coroutine(Self::shoot_coroutine));

fn update_incapacitated(node: &mut RefMut<Player>, dt: f32) {
    node.incapacitated_timer += dt;
    if node.incapacitated_timer >= node.incapacitated_duration {
        node.incapacitated_timer = 0.0;
        node.state_machine.set_state(Player::ST_NORMAL);
    }
}

fn incapacitated_coroutine(node: &mut RefMut<Player>) -> Coroutine {
    start_coroutine(async move { /* ... */ })
}

// Check the state
if self.state_machine.state() != Self::ST_DEATH {
    // Replace the current state (if ready) or next state (if in use)
    self.state_machine.set_state(Self::ST_DEATH);
}

// Used by Fight Fight in Player#fixed_update(); documented as `A hack to update a state machine being
// part of an updating struct`.
StateMachine::update_detached(&mut node, |node| &mut node.state_machine);

// Seems to be the base update API; fails if the machine is in use
node.state_machine.update(&mut player)
```
