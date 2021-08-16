# Macroquad

- [Macroquad](#macroquad)
  - [Concepts to review, from Fish Game](#concepts-to-review-from-fish-game)
  - [Hello world](#hello-world)
  - [Draw](#draw)
    - [Text](#text)
    - [Images/Sprites](#imagessprites)
    - [Animations](#animations)
    - [Tiles](#tiles)
  - [Resources handling/ECS](#resources-handlingecs)
  - [Audio/Sound](#audiosound)
  - [Input](#input)
  - [Misc](#misc)
    - [I/O](#io)
    - [Random](#random)

## Concepts to review, from Fish Game

- `macroquad-tiled`
  - `Map`
- `macroquad-platformer`
  - `World`
    - `set_actor_position`
  - `StaticTiledLayer`
  - `Solid`
  - `Collider`
  - `Actor`
- `scene`
  - `get_node(<Handle<T>>)`
  - `find_node_by_type(<T>)`
  - `add_node`
  - `node.handle`
- `ui::root_ui` -> seems GUI stuff; convenient for mid-screens, credits etc.
- `StateMachine`
  - `add_state(Self::ST_SHOOT, State::new().update(Self::update_shoot).coroutine(Self::shoot_coroutine)`
  - `set_state`
  - `update_detached(&mut node, |node| &mut node.state_machine);`
- `AnimatedSprite`
  - `set_animation`
  - `set_frame`
- `DrawTextureParams` has a `flip_<axis>`

- [`nakama`](https://github.com/heroiclabs/nakama-rs): networking (Nakama protocol)
- `nanoserde`: dependency-free serde

- `EmittersCache` -> particles

## Hello world

```rs
// All the functions/constants are in the prelude.
//
use macroquad::prelude::*;

fn window_conf() -> Conf {
    // Defaults
    Conf {
        window_title: "".into(),
        window_width: 800,
        window_height: 600,
        window_resizable: true,
        sample_count: 1,          // MSAA
        high_dpi: false,
        ..Default::default()
    }
}

#[macroquad::main("BasicShapes")]
async fn main() -> Result<(), Box<dyn error::Error>> {
    loop {
        // blah...
        next_frame().await
    }
}
```

## Draw

```rs
clear_background(RED);

draw_line(40.0, 40.0, 100.0, 200.0, 15.0, BLUE);
draw_rectangle(screen_width() / 2.0 - 60.0, 100.0, 120.0, 60.0, GREEN);
draw_rectangle_lines(screen_width() / 2.0 - 60.0, 100.0, 120.0, 60.0, 5., RED);
draw_circle(screen_width() - 30.0, screen_height() - 30.0, 15.0, YELLOW);
```

### Text

```rs
draw_text("IT WORKS!", 20.0, 20.0, 30.0, DARKGRAY);
```

### Images/Sprites

See the [resources handling](#resources-handlingecs) section for storage/load.

```rs
let sprite: Texture2D = load_texture("assets/Whale/Whale(76x66)(Orange).png").await?;

// The `draw_texture()` function has no texture params.
draw_texture_ex(
    sprite,                                          // texture
    0., 0.,                                          // x, y
    WHITE,                                           // color
    DrawTextureParams {                              // params
        source: Some(Rect::new(0.0, 0.0, 76., 66.)), // source (eg. from a sprite sheet)
        ..Default::default()
    },
);
```

### Animations

AnimationSprite doesn't have a notion of texture; it's simply has the logic to generate the coordinates inside the sprite sheet at a given point in time.

```rs
let cannon_sprite = AnimatedSprite::new(
    CANNON_WIDTH, CANNON_HEIGHT, // set the dimension of a single frame
    &[Animation { name: "idle", row: 0, frames: 1, fps: 1 }],
    &[Animation { name: "firing", row: 1, frames: 4, fps: 8 }],
    true, // `playing`: set true to start animating directly, or set separately
);

cannon_sprite.set_animation(1);
cannon_sprite.set_frame(0); // not required; 0 is default

// Put both inside draw().
//
cannon_sprite.update();
draw_texture_ex(
    cannonball_texture, // texture is separate!
    // ...
    DrawTextureParams {
        source: Some(cannonball_sprite.frame().source_rect),
        dest_size: Some(cannonball_sprite.frame().dest_size),
        flip_x: false,
        rotation: 0.0,
        ..Default::default()
    },
);
```

### Tiles

The project includes the `macroquad-tiled` crate, which supports the [Tiled](https://www.mapeditor.org) map editor.

```rs
const LAYER: &str = "main layer"; // Name in `assets/map.json`

// load map and textures [files] here ...

let tiled_map: Map = macroquad_tiled::load_map(
    &tiled_map_json,                                                // data
    &[("tileset.png", tileset), ("decorations1.png", decorations)], // textures
    &[],                                                            // external_tilesets
).unwrap(); // not compatible with `Box<dyn Error>`

tiled_map.draw_tiles(
    LAYER,
    Rect::new(0.0, 0.0, screen_width(), screen_height()), // dest
    None,                                                 // source
);

let tile: &Option<Tile> = tiled_map.get_tile(LAYER, x_u32, y_u32);

let tiles: TilesIterator = tiled_map.tiles();
```

## Resources handling/ECS

Resources are typically stored and loaded in the global storage; this is also because they're async, so they can't be loaded in a standard initializer or so.

```rs
pub struct Resources {
    pub title_texture: Texture2D,
}

impl Resources {
    pub async fn new() -> Result<Resources, Box<dyn error::Error>> {
        Ok(Resources { title_texture: load_texture("resources/cavern/title.png").await? })
    }
}

async fn main() -> Result<(), Box<dyn error::Error>> {
    let resources_loading = start_coroutine(async move {
        let resources = Resources::new().await.unwrap();
        storage::store(resources);
    });

    while !resources_loading.is_done() {
        clear_background(BLACK);
        let text = format!("Loading resources {}", ".".repeat(((get_time() * 2.) as usize) % 4));
        draw_text(text, screen_width() / 2. - 160., screen_height() / 2., 40., WHITE);
        next_frame().await;
    }
}

// In order to have a consistent assets/resource location across O/Ss, use:
set_pc_assets_folder("assets");

let resources = storage::get::<Resources>(); // get_mut() also availale
let title_texture = resources.title_texture;
```

## Audio/Sound

Formats supported out of the box: `wav`, `ogg`.

```rust
let sound1 = audio::load_sound("sound.wav").await?;
audio::play_sound_once(sound1);

audio::set_sound_volume(sound, volume_f32);
audio::play_sound(sound, PlaySoundParams { looped: true, volume: 0.3});
audio::stop_sound(sound);
```

## Input

State concepts:

- `down`    : pressed, unconditionally
- `pressed` : state changed not_pressed -> pressed

```rs
if is_key_down(KeyCode::Right) { /* ... */ }
if is_key_pressed(KeyCode::Right) { /* ... */ }
```

## Misc

### I/O

```rs
let file = load_file("path/to/file").await?
// Lossily load a string
let tiled_map_json = load_string("assets/map.json").await?;
```

### Random

```rs
// WATCH OUT!! Must seed first
rand::srand(SystemTime::now().duration_since(UNIX_EPOCH).unwrap().as_nanos() as u64);

let r: u32 = rand();                   // only for u32 ...
let r: f32 = gen_range(0., 1.)         // all numerical types; interval open on the right
let b: bool = gen_range(0., 1.) < 0.5  // bool


// Vector functionalities
//
use macroquad::rand::ChooseRandom;
vec.shuffle();                          // shuffle a vec
vec.choose()?;                          // random element
vec.choose_mut()?;                      // random element (immutable)
vec.choose_mut()?;                      // random element (mutable)
let iter = vec.choose_multiple(amount); // multiple random elements

// Generate a value in the interval [min, max).
//
let rand = gen_range(min, max);
```
