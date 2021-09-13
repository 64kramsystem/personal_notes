# Tetra

- [Tetra](#tetra)
  - [Hello world](#hello-world)
  - [Drawing](#drawing)
    - [Textures](#textures)
      - [Animations](#animations)
      - [Nineslice](#nineslice)
    - [Text](#text)
    - [Misc](#misc)
  - [Scaler/Camera](#scalercamera)
  - [Input/Events](#inputevents)
  - [Window](#window)
  - [Colliders](#colliders)
  - [Algebra (uses `vek` crate)](#algebra-uses-vek-crate)
  - [Legion "EC-without-S" integration](#legion-ec-without-s-integration)

## Hello world

```rs
// edited

struct GameState {
    paddle_texture: Texture,
    paddle_position: Vec2<f32>,
}

impl GameState {
    fn new(ctx: &mut Context) -> tetra::Result<GameState> {
        Ok(GameState {
            paddle_texture: Texture::new(ctx, PADDLE_IMAGE_PATH)?,
            paddle_position: Vec2::new(16.0, (WINDOW_HEIGHT - paddle_texture.height() as f32) / 2.),
        })
    }
}

impl State for GameState {
    fn update(&mut self, ctx: &mut Context) -> tetra::Result {
        if input::is_key_down(ctx, Key::W) {
            self.paddle_position.y -= PADDLE_SPEED;
        }
    }

    fn draw(&mut self, ctx: &mut Context) -> tetra::Result {
        graphics::clear(ctx, Color::rgb(0.3, 0.5, 0.9));
        self.paddle_texture.draw(ctx, self.paddle_position);
    }
}

fn main() -> tetra::Result {
    ContextBuilder::new("Pong", WINDOW_WIDTH as i32, WINDOW_HEIGHT as i32)
        .quit_on_escape(true)
        .build()?
        .run(GameState::new)
}
```

## Drawing

`Vec2` implementes `Into<DrawParams>`, so it can be used in place of it.

All the draw-related invocations are implied to be in the `draw()` function.

### Textures

```rs
// The image path is relative to the current (run) dir. There is (currently) no way to specify asset path(s).
//
let texture = Texture::new(ctx, PADDLE_IMAGE_PATH)?;

texture.width(), texture.height();

texture.draw(
    ctx,
    DrawParams::new()
        .position(Vec2::new(32., 32.))
        .origin(Vec2::new(8., 8.))
        .scale(Vec2::new(2., 2.))
        .color(COLOR)
        .rotation(0.5)
);

// Textures are cheap to clone, so can do liberally.
let tex2 = texture.clone();
```

#### Animations

```rs
Animation::new(
    texture, // owns the texture!
    Rectangle::row(x, y, width, height).take(frames).collect(),
    Duration::from_secs_f32(0.1),
),

/* draw() */
animation.advance(ctx);
animation.draw(ctx, DrawParams::new());
```

#### Nineslice

Seems it has limited support, only stretching:

```rs
// The first param is the full image size; the second is the border width
NineSlice::with_border(Rectangle::new(0.0, 0.0, 32.0, 32.0), 4.0);

texture.draw_nine_slice(ctx, &nineslice, 640.0, 480.0, DrawParams::new());
```

### Text

```rs
let vector_text = Text::new("TTF font", Font::vector(ctx, TTF_FILENAME, size)?);
let bitmap_text = Text::new("FNT font", Font::bmfont(ctx, FNT_FILENAME)?);

text.set_content("Hello text world!");

text.draw(ctx, DrawParams::new()); // or Vec2
```

### Misc

```rs
graphics::clear(ctx, Color::rgb(0.3, 0.5, 0.9)); // Clear screen
graphics::set_default_filter_mode(ctx, FilterMode::Linear); // Set default texture filtering (alt.: `Nearest`)
ctx_builder.timestep(Timestep::Fixed(5.0)); // set the timestep (alt.: `Variable`)
```

## Scaler/Camera

Scaling modes:

- `ShowAll`: fits to window; keep ratio
- `ShowAllPixelPerfect`: same as `ShowAll`, but scales only by integers (>= 1)
- (other)

Scaler:

```rs
// The specified values are the inner size; the outer size matches the window
let scaler = scaler: ScreenScaler::with_window_size(ctx, 640, 480, ScalingMode::ShowAllPixelPerfect)?,

/* update() */
// As of v0.6.5, it's not possible to set the scaler filtering for the scaler canvas only, as the canvas
// public reference is not mutable.
scaler.mouse_position(); // get mouse position in screen coordinates
scaler.set_mode(mode);

/* draw() */
graphics::set_canvas(ctx, scaler.canvas());
graphics::clear(ctx, Color::rgb(0.392, 0.584, 0.929));
// Draw scaled objects in this point
graphics::reset_canvas(ctx);
// Make sure that previous screen is fully cleared
graphics::clear(ctx, Color::BLACK);
scaler.draw(ctx);
```

Camera:

```rs
// The camera's viewport size should match the rendering target, in this case the ScreenScaler
let camera = Camera::new(640., 480.);

/* input() */
if let Event::Resized { width, height } = event {
    scaler.set_outer_size(width, height);
}

/* update() */
camera.position.y -= MOVEMENT_SPEED;
camera.update();

/* draw(); see scaler for some notes */
graphics::set_transform_matrix(ctx, camera.as_matrix());
// Draw transformed objects in this point
graphics::reset_transform_matrix(ctx);
// Draw untrasformed objects in this point
graphics::reset_canvas(ctx);
graphics::clear(ctx, Color::BLACK);
scaler.draw(ctx);
```

## Input/Events

```rs
input::is_key_down(ctx, Key::W);    // hold
input::is_key_pressed(ctx, Key::W); // tap
input::is_mouse_scrolled_up(ctx);
input::is_gamepad_button_down(ctx, 0, GamepadButton::Up);

/* Pad */
// Digital: (Left/Right)Shoulder, A/B/X/Y, Up/Down/Left/Right, Start/Back/Guide
// Analog: (Left/Right)Stick, (Left/Right)Trigger

input::is_gamepad_connected(ctx, 0);
input::get_gamepad_name(ctx, 0).unwrap();
let bp: Iterator<Item = &GamepadButton> = input::get_gamepad_buttons_pressed(ctx, 0);

// This works also for analog inputs
input::is_gamepad_button_down(ctx, 0, GamepadButton::LeftShoulder);
let pos: Vec2<f32> = input::get_gamepad_stick_position(ctx, 0, GamepadStick::LeftStick);
// This works for Stick/Trigger; Stick has Left/Right and X/Y
input::get_gamepad_axis_position(ctx, 0, GamepadAxis::LeftStickX),;
input::get_gamepad_axis_position(ctx, 0, GamepadAxis::LeftTrigger);

input::start_gamepad_vibration(ctx, 0, 1., 100); // strength, duration (ms)

if let Event::Resized { width, height } = event { /* ... */ }
```

## Window

```rs
// Exit (put in update())
window::quit(ctx);
```

## Colliders

```rs
// Only rectangle is provided
let bounds = Rectangle::new(x, y, width, height);
if bounds.intersects(&other_bounds).is_some() { /* ... */ }
```

## Algebra (uses `vek` crate)

```rs
// Linear interpolation
Vec2::lerp(prev_position, curr_position, blend_factor),
```

## Legion "EC-without-S" integration

Example Bunnymark, with Legion, but without the systems (edited code):

```rs
pub struct Position(pub Vec2<f32>);
pub struct Velocity(pub Vec2<f32>);

struct GameState {
    world: World,
    resources: Resources,
    texture: Texture,
}

impl GameState {
    fn new(ctx: &mut Context) -> tetra::Result<Self> {
        let mut resources = Resources::default();
        resources.insert(StdRng::from_entropy());

        Ok(GameState {
            world: World::default(),
            resources,
            texture: // ...
        })
    }

    fn spawn_bunny(&mut self) {
        let mut rng = self.resources.get_mut::<StdRng>().unwrap();
        let x_vel, y_vel = // ...

        self.world.push(
          (
            Position(Vec2::zero()),
            Velocity(Vec2::new(x_vel, y_vel))
          )
        );
    }
}

impl State for GameState {
    fn update(&mut self, ctx: &mut Context) -> tetra::Result {
        let mut query = <(&mut Velocity, &mut Position)>::query();

        // vel/pos are &mut
        for (Velocity(vel), Position(pos)) in query.iter_mut(&mut self.world) {
            *pos += *vel;

            // many computations omitted

            let mut rng = self.resources.get_mut::<StdRng>().unwrap();
            vel.y -= 3.0 + (rng.gen::<f32>() * 4.0);
        }
    }

    fn draw(&mut self, ctx: &mut Context) -> tetra::Result {
        graphics::clear(ctx, Color::rgb(0.392, 0.584, 0.929));

        let mut bunnies = <&Position>::query();

        for position in bunnies.iter(&self.world) {
            self.texture.draw(ctx, position.0);
        }
    }

    fn event(&mut self, _ctx: &mut Context, event: Event) -> tetra::Result {
        if let Event::KeyPressed { .. } = event {
            self.spawn_bunny();
        }
    }
}

fn main() -> tetra::Result {
    ContextBuilder::new("Tetra with \"EC-without-S\"", 1280, 720).quit_on_escape(true).build()?.run(GameState::new)
}
```
