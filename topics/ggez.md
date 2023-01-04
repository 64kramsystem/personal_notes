# ggez

- [ggez](#ggez)
  - [Hello world](#hello-world)
  - [General design](#general-design)
  - [Draw](#draw)
    - [Text](#text)
    - [Meshes (Geometric shapes)](#meshes-geometric-shapes)
    - [Images](#images)
    - [Sprites batching](#sprites-batching)
  - [Screen](#screen)
    - [Graphics configuration](#graphics-configuration)
    - [Viewport (/add borders)](#viewport-add-borders)
  - [Audio/Sound](#audiosound)
  - [Timing](#timing)
  - [Events Handling](#events-handling)
    - [Input](#input)
      - [Pad helpers](#pad-helpers)
    - [Window](#window)
  - [Misc](#misc)
    - [Graphics(-related) info](#graphics-related-info)
    - [Files](#files)
    - [Logging](#logging)
    - [Algebra](#algebra)

## Hello world

```rust
use ggez::event;
use ggez::graphics;
use ggez::{Context, GameResult};
use glam::Vec2;                  // Add `glam = {version = "...", features = ["mint"]}` dependency to the manifest

struct MainState {
    objects_x: f32,
    image: graphics::Image,
}

impl MainState {
    fn new(ctx: &mut Context) -> GameResult<MainState> {
        // Objects can also be instantiated inside draw(), where `&mut Context` is also available.

        // Linear filter is applied if not specified.
        //
        let mut image = graphics::Image::from_path(ctx, "/shot.png")?;
        image.set_filter(graphics::FilterMode::Nearest);

        Ok(MainState { objects_x: 0.0, image })
    }
}

impl event::EventHandler for MainState {
    fn update(&mut self, _ctx: &mut Context) -> GameResult {
        self.objects_x = self.objects_x % 800.0 + 1.0;

        Ok(())
    }

    fn draw(&mut self, ctx: &mut Context) -> GameResult {
        // Get the canvas, and clear the screen with the given (background) color.
        // WATCH OUT! Not batched; for that, must use ggez::graphics::InstanceArray.
        //
        let mut canvas = graphics::Canvas::from_frame(ctx, graphics::Color::BLACK);

        // (0, 0) is the top left.

        // Simple drawing, without transformations.
        //
        canvas.draw(&self.image, (Vec2::new(self.objects_x, 0.),));

        // Drawing with transformations.
        // Linear scaling is applied by default.
        // If there are no transformations, and the meshes have a position, one can pass `DrawParam::new()`
        // bare.
        //
        canvas.draw(
            &self.image,
            DrawParam::new()
                .dest(Vec2::new(self.objects_x, 0.))
                .rotation(0.5)
                .scale(Vec2::new(10., 10.)),
        );

        // Tell the graphics system to actually put everything on the screen. Must be called at the
        // end of draw().
        canvas.finish(context)?;

        timer::yield_now();

        Ok(())
    }
}

pub fn main() -> GameResult {
    let (mut ctx, event_loop) = ggez::ContextBuilder::new("game_id", "author")
        .window_setup(ggez::conf::WindowSetup::default().title("Snake!"))
        .window_mode(ggez::conf::WindowMode::default().dimensions(800, 600))
        .add_resource_path("./resources")
        .build()?;
    let state = MainState::new(&mut ctx)?;
    event::run(ctx, event_loop, state)
}
```

## General design

Generally speaking:

- `MainState` is the world state
  - it includes the entities
- Both `MainState` and the entities include `update()` and `draw()`
  - when the methods are called on `MainState`, it calls them on the entities

## Draw

### Text

```rust
// Coordinates: top left.
//
let font = graphics::Font::new(ctx, "/LiberationMono-Regular.ttf")?;
let text = graphics::Text::new(("Hello world!", font, 48.0));
```

### Meshes (Geometric shapes)

```rust
// The type is `Mesh` for all.

// Coordinates: center.
//
graphics::Mesh::new_circle(
    ctx,
    graphics::DrawMode::fill(),
    Vec2::new(0.0, 0.0),
    100.0,
    2.0,
    Color::WHITE,
)?;

// Filled rectangle.
//
graphics::Mesh::new_rectangle(
    ctx,
    graphics::DrawMode::fill(),
    graphics::Rect::new(450.0, 450.0, 50.0, 50.0),
    Color::WHITE,
)?;

// Stroked rectangle (with thick borders).
// Drawing this on top of the previous yields a filled rectangle with thick borders.
//
let stroked_rect = graphics::Mesh::new_rectangle(
    ctx,
    graphics::DrawMode::stroke(1.0),
    graphics::Rect::new(450.0, 450.0, 50.0, 50.0),
    graphics::Color::new(1.0, 0.0, 0.0, 1.0),
)?;
```

Compose meshes, via `MeshBuilder`:

```rust
// Meshes can also be added by calling the methods individually on the MeshBuilder instance.

let mesh = graphics::MeshBuilder::new()
    .line(
        &[Vec2::new(200.0, 200.0), Vec2::new(400.0, 200.0), Vec2::new(400.0, 400.0)], // points
        4.0, // width
        Color::new(1.0, 0.0, 0.0, 1.0),
    )?
    .circle(
        DrawMode::fill(),
        Vec2::new(600.0, 380.0),
        40.0, 1.0, // radius, tolerance
        Color::new(1.0, 0.0, 1.0, 1.0),
    )?
    .build(ctx)?;
```

Perform batch instantiation and modification, via `MeshBatch`:

```rust
// In new() ////////////////////////////////////////////////////////////////////

let mesh = graphics::Mesh::new_rectangle(
    ctx,
    graphics::DrawMode::fill(),
    graphics::Rect::new(450.0, 450.0, 50.0, 50.0),
    Color::WHITE,
)?;

let mut mesh_batch = graphics::MeshBatch::new(mesh)?;

for x in 0..100 {
    mesh_batch.add(
        DrawParam::new()
            .dest(Vec2::new(x * 16.0, 100))
            .rotation(x * PI),
    );
}

// In update() /////////////////////////////////////////////////////////////////

let instances: &mut[DrawParam] = self.mesh_batch.get_instance_params_mut();

for instance in instances.iter_mut().take(50) {
    if let graphics::Transform::Values { ref mut rotation, .. } = instance.trans {
        *rotation += 0.001 * PI * delta_time;
    }
}

// In draw() ///////////////////////////////////////////////////////////////////

// Must flush before drawing! We modified 50, so we flush that number, starting from index 0.
//
self.mesh_batch.flush_range(ctx, graphics::MeshIdx(0), 50)?;
self.mesh_batch.draw(ctx, DrawParam::default())?;
```

Textured triangle:

```rust
let mb = &mut graphics::MeshBuilder::new();
let triangle_verts = vec![
    graphics::Vertex {
        pos: [100.0, 100.0],
        uv: [1.0, 1.0],
        color: [1.0, 0.0, 0.0, 1.0],
    },
    graphics::Vertex {
        pos: [0.0, 100.0],
        uv: [0.0, 1.0],
        color: [0.0, 1.0, 0.0, 1.0],
    },
    graphics::Vertex {
        pos: [0.0, 0.0],
        uv: [0.0, 0.0],
        color: [0.0, 0.0, 1.0, 1.0],
    },
];

let triangle_indices = vec![0, 1, 2];

let image = graphics::Image::from_path(ctx, "/rock.png")?;
mb.raw(&triangle_verts, &triangle_indices, Some(image))?;
mb.build(ctx)
```

### Images

Coordinates: top left.

See [Hello world](#hello-world).

### Sprites batching

Draw images in batch - for large amounts, it saves considerable time (bunnymark (1000 images) is ~8x as fast).

```rust
// In new() ////////////////////////////////////////////////////////////////////

let bunnybatch = SpriteBatch::new(image);

// In draw() ///////////////////////////////////////////////////////////////////

self.bunnybatch.clear();

for bunny in &self.bunnies {
    self.bunnybatch.add((bunny.position,));
}

graphics::draw(ctx, &self.bunnybatch, (Vec2::new(0.0, 0.0),))?;
```

## Screen

It's possible to draw to an arbitrary destination; see `render_to_image.rs`.

### Graphics configuration

Fullscreen/windowed, and resizability:

```rust
// From ContextBuilder

enum conf::FullscreenType {
    Windowed,
    True,     // also it allows us to set different resolutions (may not work well)
    Desktop,  // modern preference; plays nicer with multiple monitors.
}

context_builder
    // watch out, this is not .window_setup()!
    .window_mode(
        conf::WindowMode::default()
            .fullscreen_type(conf::FullscreenType::Desktop)
            .vsync(true) // default: true
            .resizable(true)
    )

// From Window

enum ggez::winit::window::Fullscreen {
    Exclusive(VideoMode),
    /// Providing `None` to `Borderless` will fullscreen on the current monitor.
    Borderless(Option<MonitorHandle>),
}

context.gfx.window().set_fullscreen(Some(Fullscreen::Borderless(None)))
```

Antialiasing:

```rust
// Supported: 2⁰ to 2⁴
//
context_builder
    .window_setup(
        // or: `conf::NumSamples::Four`
        conf::WindowSetup::default().samples(conf::NumSamples::try_from(4)?)
    )
```

Backend:

```rust
enum Backend {
    // Defaults to 3.2, which is supported by basically every machine since 2009 or so
    OpenGL { major: u8, minor: u8 },
    // Defaults to 3.0, used for mobile devices; older versions have limitations.
    #[allow(clippy::upper_case_acronyms)]
    OpenGLES { major: u8, minor: u8 },
}

let backend = conf::Backend::default(); // OpenGL
let backend = conf::Backend::OpenGLES { major: 3, minor: 0 };

context_builder
    .backend(backend)
```

### Viewport (/add borders)

```rs
// Assume that `fullscreen_type` is set to `conf::FullscreenType::Desktop`.

// Returns the viewport and scissor coordinates.
//
fn compute_viewport(context: &Context) -> (Rect, Rect) {
    // Assume that the pixels are square.
    //
    let PhysicalSize {
        width: screen_width,
        height: screen_height,
    } = context.gfx.window().inner_size();

    let game_ratio = WINDOW_WIDTH / WINDOW_HEIGHT;
    let screen_ratio = screen_width as f32 / screen_height as f32;

    let (viewport_width, viewport_height, scaling_ratio) = if screen_ratio >= game_ratio {
        (
            WINDOW_HEIGHT * screen_ratio,
            WINDOW_HEIGHT,
            screen_height as f32 / WINDOW_HEIGHT,
        )
    } else {
        (
            WINDOW_WIDTH,
            WINDOW_WIDTH / screen_ratio,
            screen_width as f32 / WINDOW_WIDTH,
        )
    };

    let tot_border_width = viewport_width - WINDOW_WIDTH;
    let tot_border_height = viewport_height - WINDOW_HEIGHT;

    let viewport_rect = Rect::new(
        -tot_border_width / 2.,
        -tot_border_height / 2.,
        viewport_width,
        viewport_height,
    );

    let scissors_rect = Rect::new(
        (screen_width as f32 - WINDOW_WIDTH * scaling_ratio) / 2.,
        (screen_height as f32 - WINDOW_HEIGHT * scaling_ratio) / 2.,
        viewport_rect.w * scaling_ratio,
        viewport_rect.h * scaling_ratio,
    );

    (viewport_rect, scissors_rect)
}

// Then before drawing, set the canvas viewport and the scissor:

canvas.set_screen_coordinates(self.viewport_rect);
canvas.set_scissor_rect(self.scissors_rect)?;
```

It's also possible to draw upside down:

```rust
// This is an upside-down 800x600 viewport; note the second Y value.
//
let viewport = graphics::Rect::new(0.0, 600.0, 800.0, -600.0);
graphics::set_screen_coordinates(ctx, viewport).unwrap();
```

Since the event coordinates are absolute, if the coordinates system (viewport) is changed, the logical coordinates need to be computed:

```rust
let screen_rect = graphics::screen_coordinates(ctx);
let size = graphics::window(ctx).inner_size(); // seems the same as graphics::drawable_size(ctx)
logical_x = (x / (size.width  as f32)) * screen_rect.w + screen_rect.x;
logical_y = (y / (size.height as f32)) * screen_rect.h + screen_rect.y;
```

## Audio/Sound

Don't forget that the reference must be in scope, somewhere :)

```rust
let sound = audio::Source::new(ctx, "/pew.ogg")?;
sount.set_volume(0.5);

// Playback operations (it's correct that some methods require Context, while others don't)

sound.play_detached(ctx)?;     // Play asynchronously (even if dropped), without wait; can't be stopped
sound.play_later()?;           // Queue playback, waiting each time for the sound to finish
sound.play(ctx).unwrap();      // Shortcut for stop() + play_later()
sound.set_repeat(true);
sound.pause();
sound.resume();
sound.stop(ctx)?;

// Example queries

while sound.playing() { println!("Elapsed time: {:?}", sound.elapsed()) }
sound.stopped();
sound.volume();
sound.repeat();

// Example effects

sound.set_pitch(2.0);
```

## Timing

The internal timer is in the `TimeContext` type, which includes a `residual_update_dt`.

```rust
// Returns true if the time elapsed since the last check is higher than the target FPS.
// If more than a frame passed, is subtracts one frame time from `residual_update_dt`.
//
// This pattern is used to run the intended number of updates per second: if the time is early, no cycles
// are run, otherwise, one cycle is performed for each cycle fitting in the time since the last update.
//
while timer::check_update_time(ctx, UPDATES_PER_SECOND) {
    self.rotation += 0.01;
}

// The game doesn't implement a frame limiter; it requires yielding (via framework, although it's just
// forwarded to the O/S).
//
fn draw(&mut self, ctx: &mut Context) -> GameResult {
  // blah
  timer::yield_now();
  Ok()
}
```

## Events Handling

### Input

Example for a game where keeping a key pressed is meaningful; in a game where this doesn't apply, `key_up_event()` can be ignored.

```rust
impl EventHandler for MainState {
    fn key_down_event(&mut self, _ctx: &mut Context, keycode: VirtualKeyCode, _keymod: KeyMods, _repeat: bool) {
        match keycode {
            VirtualKeyCode::Up => { self.input.yaxis = 1.0; }
            VirtualKeyCode::Left => { self.input.xaxis = -1.0; }
            VirtualKeyCode::Right => { self.input.xaxis = 1.0; }
            VirtualKeyCode::Space => { self.input.fire = true; }
            VirtualKeyCode::Escape => ctx.request_quit(),            // Exit API
            _ => {}
        }
    }

    fn key_up_event(&mut self, _ctx: &mut Context, keycode: VirtualKeyCode, _keymod: KeyMods) {
        match keycode {
            VirtualKeyCode::Up => { self.input.yaxis = 0.0; }
            VirtualKeyCode::Left | VirtualKeyCode::Right => { self.input.xaxis = 0.0; }
            VirtualKeyCode::Space => { self.input.fire = false; }
            _ => {}
        }
    }

    // Text input (simplified handling); called after (each) key_down event.
    //
    fn text_input_event(&mut self, _ctx: &mut Context, ch: char) { /* ... */ }
}

// Key status can be checked from the context:

let a_key_pressed: bool = context.keyboard.is_key_pressed(VirtualKeyCode::A);
let shift_key_pressed: bool =  context.keyboard.is_mod_active(keyboard::KeyMods::SHIFT)
let keys_pressed: &HashSet<VirtualKeyCode> = context.keyboard.pressed_keys();

// Repetition test; returns true as long as the key is pressed.
// Seems not to work correctly; on a test loop with is_key_pressed + is_key_repeated, it took around
// 20 loops for the latter to start returning true.
let key_kept_pressed: bool = context.keyboard.is_key_repeated();
```

Mouse:

```rust
fn mouse_button_down_event(&mut self, _ctx: &mut Context, button: input::mouse::MouseButton, x: f32, y: f32) {
    if button == input::mouse::MouseButton::Left { /* .. */ }
}

fn mouse_button_up_event(&mut self, _ctx: &mut Context, button: MouseButton, x: f32, y: f32) { /* ... */ }
fn mouse_motion_event(&mut self, _ctx: &mut Context, x: f32, y: f32, xrel: f32, yrel: f32) { /* ... */ }
fn mouse_wheel_event(&mut self, _ctx: &mut Context, x: f32, y: f32) { /* ... */ }
```

Pad:

```rust
pub fn pad_input(context: &Context, pad_number: PadNum, test: fn(&Gamepad) -> bool) -> bool {
    let mut pad_iter = context.gamepad.gamepads();

    let pad = match pad_number {
        PadNum::Zero => pad_iter.next(),
        PadNum::One => pad_iter.nth(1),
    };

    match pad {
        None => false,
        Some((_id, pad)) => test(&pad),
    }
}

// Pad layout: https://docs.rs/ggez/latest/ggez/event/enum.Button.html

fn gamepad_button_down_event(&mut self, _ctx: &mut Context, btn: Button, id: GamepadId) { /* .. */ }
fn gamepad_button_up_event(&mut self, _ctx: &mut Context, btn: Button, id: GamepadId) { /* .. */ }
fn gamepad_axis_event(&mut self, _ctx: &mut Context, axis: Axis, value: f32, id: GamepadId) { /* .. */ }
```

#### Pad helpers

```rs
pub enum PadNum { Zero, One }
pub const ANALOG_STICK_TOLERANCE: f32 = 0.1;

pub fn pad_input(context: &Context, pad_number: PadNum, test: fn(&Gamepad) -> bool) -> bool {
    let mut pad_iter = gamepad::gamepads(context);
    let pad = match pad_number {
        PadNum::Zero => pad_iter.next(),
        PadNum::One => pad_iter.skip(1).next(),
    };
    pad.is_some_and(|(_id, pad)| test(pad))
}

pub fn is_pad_up_pressed(context: &Context, pad_number: PadNum) -> bool {
    pad_input(context, pad_number, |pad| { pad.value(Axis::LeftStickY) > ANALOG_STICK_TOLERANCE })
}

pub fn is_fire_button_pressed(context: &Context, pad_number: PadNum) -> bool {
    // Oddly, on two pads tested, X was mapped to a different button, so we catch both.
    //
    pad_input(context, pad_number, |pad| { pad.is_pressed(Button::West) || pad.is_pressed(Button::North) })
}
```

### Window

```rust
fn resize_event(&mut self, ctx: &mut Context, new_width: f32, new_height: f32) { /* ... */ }

fn focus_event(&mut self, _ctx: &mut Context, gained: bool) { /* gained/lost */ }
```

## Misc

### Graphics(-related) info

Window functions return zeros if the window doesn't exist.

```rust
// Graphics system info (drivers etc.).
//
let info: String = graphics::renderer_info(&ctx)?;

// Canvas dimensions.
//
let (width, height) = graphics::drawable_size(ctx);

// Size of the window, including borders, titlebar, etc.
//
let (width, height) = graphics::size(ctx);

// Averaged from the last 200 frames.
//
let fps: f64 = ggez::timer::fps(ctx);
```

### Files

The filesystem library implements a VFS.

```rust
// Print context filesystem info
filesystem::print_all(ctx);

// Get files of a dir
let files: Box<dyn Iterator<Item = path::PathBuf>> = filesystem::read_dir(ctx, "/")?;

let test_file = path::Path::new("/path/to/file");

// Write
{
    let mut file = filesystem::create(ctx, test_file)?;
    file.write_all(b"foo")?;
}

// Append
{
    let options = filesystem::OpenOptions::new().append(true);
    let mut file = filesystem::open_options(ctx, test_file, options)?;
    file.write_all(b"foo")?;
}

// Read
{
    let mut buffer = Vec::new();
    let mut file = filesystem::open(ctx, test_file)?;
    file.read_to_end(&mut buffer)?;
}

// Delete
filesystem::delete(ctx, "/path/to/file")?;

// Write a configuration file `conf.toml` to the user dir.
let config = conf::Conf::new();
filesystem::write_config(ctx, &config)?;

// Read a configuration file, named `conf.toml`, in any resource dir.
let config = filesystem::read_config(ctx, )?;
```

### Logging

See `logging.rs` example.

### Algebra

ggez uses the `glam` crate:

```rust
// Related types: Mat2/3/4, Quat, Vec3/4; Vec3A is SIMD

const X_AXIS: Vec2 = const_vec2!([1., 0.]);
let vec2 = glam::Vec2::new(200., 100.);
let normalize = vec2.normalize();
```
