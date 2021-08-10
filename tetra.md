# Tetra

- [Tetra](#tetra)
  - [Hello world](#hello-world)
  - [Drawing](#drawing)
    - [Textures](#textures)
    - [Misc](#misc)
  - [Input](#input)
  - [Window](#window)
  - [Colliders](#colliders)
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
        self.paddle_texture.draw(ctx, self.paddle_position); // Vec2 -> Into<DrawParams>
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

### Textures

```rs
let texture = Texture::new(ctx, PADDLE_IMAGE_PATH)?;
texture.width(), texture.height();
```

### Misc

```rs
// Clear screen
graphics::clear(ctx, Color::rgb(0.3, 0.5, 0.9));
```

## Input

```rs
input::is_key_down(ctx, Key::W);    // hold
input::is_key_pressed(ctx, Key::W); // tap
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
