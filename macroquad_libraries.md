# Macroquad Libraries

- [Macroquad Libraries](#macroquad-libraries)
  - [Tiled (`tiled`)](#tiled-tiled)
  - [Tiles (`tiles`) + physics (`physics-platformer`), with example](#tiles-tiles--physics-physics-platformer-with-example)
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

APIs not covered in the example:

```rs
// Get tile at x/y (tile units, not pixels)
let tile: Option<&Tile> = tiled_map.get_tile(LAYER_NAME, x, y)

// Get an object world coordinates; objects are inside layers.
let objects = tiled_map.layers["logic"].objects;
let macroquad_tiled::Object { world_x, world_y, .. } = objects[0];

// Layer-related
tiled_map.layers["layer_name"].width // in tiles
```

## Tiles (`tiles`) + physics (`physics-platformer`), with example

Types:

- `World`            : layers, (solids+colliders) + (actors+colliders)
- `StaticTiledLayer` : tiles
- `Collider`
- `Actor`            : (only id)
- `Solid`            : (only id) this is a MQ concept, not Tiled
- `Tile`             : tyle type (enum)

Concepts not covered:

- squished

```rs
struct Player {
    collider: Actor,
    speed: Vec2,
}

struct Platform {
    collider: Solid,
    speed: f32,
}

async fn load_tiled_map() -> Map {
    let tileset = load_texture("resources/tileset.png").await.unwrap();
    tileset.set_filter(FilterMode::Nearest);

    let map_json = load_string("resources/map.json").await.unwrap();
    macroquad_tiled::load_map(&map_json, &[("tileset.png", tileset)], &[]).unwrap()
}

fn add_tile_colliders(map: &Map, world: &mut World) {
    // Get the tiles within a Rect (or all the tiles, if None).
    let tiles_itr = map.tiles(LAYER_NAME, None);

    // Map tile types to internal ones.
    // map.tiles
    let static_colliders = tiles_itr
        .map(|(_x, _y, tile)| match tile {
            None => Tile::Empty,
            Some(tile) if tile.attrs.contains("jumpthrough") => Tile::JumpThrough,
            _ => Tile::Solid,
        })
        .collect();

    world.add_static_tiled_layer(
        static_colliders,
        map.raw_tiled_map.tilewidth as f32,
        map.raw_tiled_map.tileheight as f32,
        map.raw_tiled_map.width as usize,
        LAYER_TAG,
    );
}

// Need to avoid collision with prelude::set_camera.
//
fn prepare_and_set_camera(map: &Map) {
    let camera_area = Rect::new(
        0.,
        0.,
        (map.layers[LAYER_NAME].width * map.raw_tiled_map.tilewidth) as f32,
        (map.layers[LAYER_NAME].height * map.raw_tiled_map.tileheight) as f32,
    );

    let camera = Camera2D::from_display_rect(camera_area);

    set_camera(&camera);
}

fn draw_tiles(map: &Map) {
    let dest_pos = Rect::new(
        0.,
        0.,
        (map.layers[LAYER_NAME].width * map.raw_tiled_map.tilewidth) as f32,
        (map.layers[LAYER_NAME].height * map.raw_tiled_map.tileheight) as f32,
    );

    map.draw_tiles(LAYER_NAME, dest_pos, None);
}

fn draw_platform(platform: &Platform, world: &World, map: &Map) {
    let width = (PLATFORM_SIZE.0 * map.raw_tiled_map.tilewidth) as f32;
    let height = (PLATFORM_SIZE.1 * map.raw_tiled_map.tileheight) as f32;

    let source_pos = Rect::new(
        (PLATFORM_SOURCE_POS.0 * map.raw_tiled_map.tilewidth) as f32,
        (PLATFORM_SOURCE_POS.1 * map.raw_tiled_map.tilewidth) as f32,
        width,
        height,
    );

    let dest_pos = Rect::new(
        world.solid_pos(platform.collider).x,
        world.solid_pos(platform.collider).y,
        width,
        height,
    );

    map.spr_ex(TILESET_NAME, source_pos, dest_pos)
}

fn draw_player(world: &World, player: &Player, map: &Map) {
    let world = world.actor_pos(player.collider);

    let width = (PLAYER_SIZE.0 * map.raw_tiled_map.tilewidth) as f32;
    let height = (PLAYER_SIZE.1 * map.raw_tiled_map.tileheight) as f32;

    // Watch out! `>= 0.0` is not a correct test, since it matches `-0.0`, which means facing left!.
    //
    let dest_pos = if player.speed.x.is_sign_positive() {
        Rect::new(world.x, world.y, width, height)
    } else {
        Rect::new(world.x + width, world.y, -width, height)
    };

    map.spr(TILESET_NAME, PLAYER_SPRITE_ID, dest_pos);
}

fn move_player(world: &mut World, player: &mut Player) {
    // Handle X movement

    if is_key_down(KeyCode::Right) {
        player.speed.x = PLAYER_SPEED;
    } else if is_key_down(KeyCode::Left) {
        player.speed.x = -PLAYER_SPEED;
    } else {
        // The sign is the facing direction, so we need to preserve it.
        //
        player.speed.x = 0. * player.speed.x.signum();
    }

    // Handle Y movement

    let actor_pos = world.actor_pos(player.collider);
    let on_ground = world.collide_check(player.collider, actor_pos + vec2(0., 1.));

    if on_ground {
        if is_key_pressed(KeyCode::Space) {
            if is_key_down(KeyCode::Down) {
                // Descend, if there is a Jumpthough tile.
                world.descent(player.collider);
            } else {
                player.speed.y = JUMP_SPEED;
            }
        }
    } else {
        player.speed.y += GRAVITY_SPEED * get_frame_time();
    }

    // Move actor

    world.move_h(player.collider, player.speed.x * get_frame_time());
    world.move_v(player.collider, player.speed.y * get_frame_time());
}

fn move_platform(world: &mut World, platform: &mut Platform) {
    world.solid_move(platform.collider, platform.speed * get_frame_time(), 0.0);

    let solid_pos = world.solid_pos(platform.collider);

    if platform.speed.is_sign_positive() && solid_pos.x >= 220. {
        platform.speed *= -1.;
    }
    if platform.speed.is_sign_negative() && solid_pos.x <= 150. {
        platform.speed *= -1.;
    }
}

#[macroquad::main("Platformer")]
async fn main() {
    let mut world = World::new();

    let map = load_tiled_map().await;

    // When a new actor is added, create a collider for it, and add it to the world.
    // Use the collsion box dimensions here (which can be different from the sprite ones)
    //
    let mut player = Player {
        collider: world.add_actor(vec2(50.0, 80.0), 8, 8),
        speed: vec2(0., 0.),
    };

    // Do the same, but for a solid.
    let mut platform = Platform {
        collider: world.add_solid(vec2(170.0, 130.0), 32, 8),
        speed: PLATFORM_SPEED,
    };

    add_tile_colliders(&map, &mut world);

    prepare_and_set_camera(&map);

    loop {
        clear_background(BLACK);

        draw_tiles(&map);

        draw_platform(&platform, &world, &map);

        draw_player(&world, &player, &map);

        move_player(&mut world, &mut player);

        move_platform(&mut world, &mut platform);

        next_frame().await
    }
}
```

Other solid-related methods (similar to Actor):

- `solid_at(&self, pos: Vec2) -> bool`
- `collide_solids(&self, pos: Vec2, width: i32, height: i32) -> Tile`

Other methods:

- `tag_at(&self, pos: Vec2, tag: u8) -> bool`: true if the pos matches: a non-empty tile on that layer, or a collidable solid
- `collide_tag(&self, tag: u8, pos: Vec2, width: i32, height: i32) -> Tile`
- `squished(&self, actor: Actor) -> bool`: true if the actor is squished


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
