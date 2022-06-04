# Fyrox

- [Fyrox](#fyrox)
  - [Hello world](#hello-world)
  - [Misc notes and concepts](#misc-notes-and-concepts)
  - [GameState methods](#gamestate-methods)
  - [Pool](#pool)
  - [Textures](#textures)
  - [Sound](#sound)
  - [Scene](#scene)
    - [Concepts/Terminology](#conceptsterminology)
    - [Graph](#graph)
    - [Sprites (2d)](#sprites-2d)
    - [Camera](#camera)
  - [Transforms](#transforms)
  - [Events](#events)
    - [Window events](#window-events)
    - [Input](#input)
      - ["Just pressed" functionality](#just-pressed-functionality)
  - [Window configuration](#window-configuration)
  - [More sophisticated hello world (2d)](#more-sophisticated-hello-world-2d)
  - [Random](#random)

Notes from the [Fyrox Cheat Book](https://fyrox-book.github.io/fyrox/introduction.html).

## Hello world

```rs
use fyrox::{engine::framework::prelude::*, engine::Engine};

struct Game {}

impl GameState for Game {
    fn init(_engine: &mut Engine) -> Self { /* ... */ }
}

fn main() {
    Framework::<Game>::new().unwrap().title("Simple").run();
}
```

## Misc notes and concepts

IMPORTANT! In order to perform blocking resource loading, use `block_on()`; all the code in this guide assumes that the dev will add it where required:

```rs
let resource = block_on(resource_manager.request_*(path)).unwrap();
```

3d:

- pitch: look up!
- yam: look side!

## GameState methods

Default methods:

```rs
use fyrox::{
    event::{DeviceEvent, DeviceId, WindowEvent},
    event_loop::ControlFlow,
    gui::message::UiMessage,
};

fn on_tick(&mut self, engine: &mut Engine, dt: f32, control_flow: &mut ControlFlow) {
    // called at fixed 60 FPS; `dt` returns the amount of seconds that passed from the previous
    // call; `control_flow` allows changing the execution flow, for example quitting.
}
fn on_ui_message(&mut self, engine: &mut Engine, message: UiMessage) {
    // handle UI events
}
fn on_device_event(&mut self, engine: &mut Engine, device_id: DeviceId, event: DeviceEvent) {
    // handle input from devices
}
fn on_window_event(&mut self, engine: &mut Engine, event: WindowEvent) {
    // window events
}
fn on_exit(&mut self, engine: &mut Engine) {
    // called on shutdown; useful for cleanup
}
```

## Pool

Generic APIs:

```rs
let pool = Pool::<String>::new();

let handle: Handle<String> = pool.spawn("str".to_string());

let str_ref: &String = pool.borrow(handle) // Borrow object; format: [try_]borrow[_mut]()
let str_ref = pool[handle]                 // borrow via Index trait
let str_own: String = pool.free(handle)    // Extract and free object (will be removed from the pool)

pool.is_valid_handle(handle);

// Iterate a pool.
//
let mut iter = pool.iter_mut();
iter.next().unwrap();          // "str" (&String)

// Iterate and filter (out) pool elements.
//
pool.retain(|s| s.start_with("*"));
```

Other APIs:

```rs
// Reservation; used to extract an object and (put it back or remove it). Reserved objects can't be
// borrowed in the meanwhile.
//
let (ticket, mut obj): Ticket<T>, T = pool.take_reserve(handle) // Format: [try_]take_reserve()
// ...mess with obj...
pool.put_back(ticket, obj)
pool.forget_ticket(ticket)                                      // Remove from the pool

let str_handle = pool.handle_of(str_ref); // Get a handle from a ref
pool.clear();                             // Destroy all the pool entries

Handle::NONE;                             // Invalid handle; very useful for placeholder handles!
```

## Textures

As of Fyrox v0.25, loading textures in debug mode is extremely slow (1.4" for each PNG file, even if small), so we need to load them asynchronously.

WATCH OUT! As of Fyrox v0.26, some PNG files display with corruption (see https://github.com/FyroxEngine/Fyrox/issues/320).

```rs
let texture_requests = join_all(
    image_paths
        .iter()
        .map(|path| resource_manager.request_texture(path)),
);

let images = image_paths
    .iter()
    .zip(block_on(texture_requests))
    .map(|(path, texture)| (path.to_string(), texture.unwrap()))
    .collect::<HashMap<_, _>>();
```

Image (texture) sizes are tricky to get:

```rs
if let TextureKind::Rectangle { width, height } = texture.data_ref().kind() { ... }
```

## Sound

Sounds must be attached to a node, otherwise, they're not played.

```rs
let sound = resource_manager.request_sound_buffer(path);

let sound_h = SoundBuilder::new(BaseBuilder::new())
    .with_buffer(Some(sound))
    .with_status(Playing)          // doesn't start automatically
    .with_gain(0.5)                // volume (0..1.0; can be set > 1.0); default = 1.0; logarithmic scale
    .build(&mut scene.graph);

builder.
  .with_looping(true)           // loop (use as music)
  .with_play_once(true)         // destroy after playback; avoids manual node removal

// Get node, use it, and remove it.
let mut music: Sound = scene.graph[music_h].as_sound_mut();
music.blah_blah();
music.stop();

// It's good practice to remove the stopped nodes (but see the play_once flag).
//
// Statuses: Stopped, Playing, Paused
//
if music.status == Status::Stopped {
  // Removing the node also stops the playback.
  //
  scene.remove_node(music_h);
}

// Add effects.
//
let reverb_effect = ReverbEffectBuilder::new(BaseEffectBuilder::new().with_gain(0.7)) // Otherwise it's too loud!
  .with_decay_time(3.0)  // Set reverb time to ~3 seconds - the longer, the deeper the echo.
  .build(&mut scene.graph.sound_context);

scene.graph
    .sound_context
    .effect_mut(reverb_effect)
    .inputs_mut()
    .push(EffectInput{
        sound: source,
        filter: None
    });
```

## Scene

Loading:

```rs
// See [example](https://github.com/FyroxEngine/Fyrox/blob/master/examples/async.rs) for async scene loading.

fn load_scene(resource_manager: ResourceManager) -> Scene {
    // Create parent scene.
    let mut parent_scene = Scene::new();

    // Request child scene and block until it loading.
    let scene_resource: Model = resource_manager.request_model("path/to/your/scene.rgs");

    // Create an instance of the scene in the parent scene.
    // This can also be, say, a car. If the car prefab is modified, the instances in all the scenes
    // are automatically updated.
    let child_scene: Handle<Node> = scene_resource.instantiate_geometry(&mut parent_scene);

    parent_scene
}
```

Other APIs:

```rs
// Freeze a scene, without needing to unload it.
//
scene.enabled = false;

// Set the ambient light; there is a bright enough default.
//
scene.ambient_lighting_color = Color::opaque(30, 30, 30);
```

### Concepts/Terminology

- **Base**: node that stores hierarchical information (a handle to the parent node and a set of handles to children
  nodes), local and global transform, name, tag, lifetime, etc. It has self-describing name - it is used as a base
  node for every other scene node (via composition).
- **Mesh**: node that represents a 3D model. This one of the most commonly used nodes in almost every game. Meshes
  could be easily created either programmatically, or be made in some 3D modelling software (like Blender) and
  loaded in your scene.
- **Light**: node that represents a light source. There are three types of light sources:
  - **Directional**: light source that does not have position, only direction. The closest real-world example is our Sun.
  - **Point**: light source that emits light in every direction. Real-world example: light bulb.
  - **Spot**: light source that emits light in a particular direction with a cone-like shape. Real-world example:
    flashlight.
- **Camera**: node that allows you to see the world. You must have at least one camera in your scene to be able to see
  anything.
- **Sprite**: node that represents a quad that always faced towards a camera. It can have a texture, size, it also can
  be rotated around the "look" axis.
- **Particle system**: node that allows you to build visual effects using a huge set of small particles, it can be used
  to create smoke, sparks, blood splatters, etc. effects.
- **Terrain**: node that allows you to create complex landscapes with minimal effort.
- **Decal**: node that paints on other nodes using a texture. It is used to simulate cracks in concrete walls, damaged
  parts of the road, blood splatters, bullet holes, etc.
- **Rigid Body (2D)**: physical entity that is responsible for dynamic of the rigid. There is a special variant for 2D -
  RigidBody2D.
- **Collider (2D)**: physical shape for a rigid body, it is responsible for contact manifold generation, without it any
  rigid body will not participate in simulation correctly, so every rigid body must have at least one collider. There
  is a special variant for 2D - Collider2D.
- **Joint (2D)**: physical entity that restricts motion between two rigid bodies, it has various amounts of degrees of
  freedom depending on the type of the joint. There is a special variant for 2D - Joint2D.
- **Rectangle**: simple rectangle mesh that can have a texture and a color, it is a very simple version of a Mesh node,
  yet it uses very optimized renderer, that allows you to render dozens of rectangles simultaneously. This node is
  intended to be used for 2D games only.
  **Pivot**: can be used as invisible node, to group other nodes

- **Prefab**: Scene used to create a source of data for other scenes.

### Graph

Nodes are (generally) created via builders:

- `BaseBuilder`
- `CameraBuilder`
- `MeshBuilder`
- `LightBuilder`
- `SpriteBuilder`
- `ParticleSystemBuilder`
- `TerrainBuilder`
- `DecalBuilder`
- `RigidBody`
- `Collider`
- `Joint`
- `Rectangle`
- `Pivot`

They all implement `new(base_builder: BaseBuilder)` (except `BaseBuilder`), due to the composition-based design. Example:

```rs
fn create_node(scene: &mut Scene) -> Handle<Node> {
    CameraBuilder::new(
        BaseBuilder::new()
            // Add some children nodes.
            .with_children(&[
                MeshBuilder::new(
                    BaseBuilder::new()
                        .with_name("Staff")
                        .with_local_transform(
                            TransformBuilder::new()
                                .with_local_position(Vector3::new(0.5, 0.5, 1.0))
                                .build(),
                        ),
                )
                .build(&mut scene.graph),
                SpriteBuilder::new(
                  // ...
                ).build(&mut scene.graph),
            ])
            .with_local_transform(
              // ...
            ),
    )
    .with_fov(60.0f32.to_radians())
    .build(&mut scene.graph)
}
```

In order to group nodes, can use a `Pivot` (which is not visible). WATCH OUT!! Scaling must be performed on the children nodes, not on the pivot.

Other operations:

```rs
// Root node.
//
scene.graph[scene.graph.get_root()]

// Add a node manually.
//
let node: Node = CameraBuilder::new(BaseBuilder::new()).build_node();
scene.graph.add_node(node)

// Attach and link a node to another.
//
// When a node is linked to another, it becomes child of both the root node and the linkee node, however,
// its properties become relative the the linkee node.
//
// Use a `builder...build(graph)` to get a Handle<Node>.
//
let weapon: Handle<Node> = resource_manager.request_model("path/to/weapon.fbx")
    .instantiate_geometry(scene);
scene.graph.link_nodes(weapon, camera);

// Remove node (including the children). If the node is linked to another, it's removed from the children
// of both nodes.
//
scene.graph.remove_node(handle)

// Remove node, without removing the children (only detach them).
//
for child in scene.graph[node_to_remove].children().to_vec() {
    scene.graph.unlink_node(child);
}
scene.graph.remove_node(node_to_remove);
```

### Sprites (2d)

Add a sprite:

```rs
// If the camera size doesn't match the pixels size, must transform the sprites.
//
let (local_x_size, local_y_size) = (0.38, 0.35);
RectangleBuilder::new(
    BaseBuilder::new().with_local_transform(
        TransformBuilder::new()
            .with_local_position(Vector3::new(x, y, 0.0))
            .with_local_scale(Vector3::new(local_x_size, local_y_size, f32::EPSILON))
            .build(),
    ),
)
.with_texture(resource_manager.request_texture("examples/data/starship.png"))
.build(&mut scene.graph);
```

### Camera

Orthographic camera:

```rs
// WATCH OUT!!!:
//
// - (0, 0) is at the center of the screen (on start)
// - (X, Y) point (left, top)
//
CameraBuilder::new(BaseBuilder::new())
    .with_projection(Projection::Orthographic(OrthographicProjection {
        z_near: -0.1,
        z_far: 16.0,
        vertical_size: 2.0, // half of the total vert. size
        // the horizontal size is automatically computed
    }))
    .build(graph)
```

On a fixed-screen 2d game, on can use the camera as root node, and attach/detach sprite nodes are required; the only transform they need is a local scale to the sprite size. It's still easier though, to use the background texture as root node.

## Transforms

```rs
// Transform a node during building
//
TransformBuilder::new()
    .with_local_position(Vector3::new(x, y, z)
    .with_local_scale(Vector3::new(sprite_local_size, sprite_local_size, f32::EPSILON))
    .build()

// Transform an existing node.
//
let local_transform = scene.graph[node].local_transform_mut();
local_transform.set_rotation(UnitQuaternion::from_euler_angles(
    180.0_f32.to_radians(), // roll
    0.0,                    // pitch
    0.0,                    // yaw
));
```

## Events

### Window events

```rs
WindowEvent::CloseRequested => *control_flow = ControlFlow::Exit,
WindowEvent::Resized(size) => {
    // Don't forget this :)
    engine.set_frame_size(size.into()).unwrap();
}
```

### Input

Base handling of keyboard and mouse; doesn't handle "just pressed" functionality:

```rs
struct InputController {
    move_forward: bool,
    move_backward: bool,
    move_left: bool,
    move_right: bool,
    pitch: f32,          // For 3d
    yaw: f32,            // ^^
}

fn on_window_event(&mut self, engine: &mut Engine, event: WindowEvent) {
    if let WindowEvent::KeyboardInput { input, .. } = event {
        if let Some(key_code) = input.virtual_keycode {
            // The logic here is for simplicity; see note above the code block.
            //
            match key_code {
                VirtualKeyCode::W => {
                    if input.state == ElementState::Pressed { /* ... */ }
                }
            }
        }
    }
}

fn on_device_event(&mut self, _engine: &mut Engine, _device_id: DeviceId, event: DeviceEvent) {
    if let DeviceEvent::MouseMotion { delta } = event {
        // Clamp the pitch, in order to prevent the camera from turning upside down.
        //
        self.controller.yaw -= delta.0;
        self.controller.pitch = (self.controller.pitch + delta.1).clamp(-90.0, 90.0);
    }
}

fn on_tick(&mut self, engine: &mut Engine, _dt: f32, _: &mut ControlFlow) {
    let mut offset = Vector3::default();
    if self.input_controller.move_forward {
        offset.y += 1.0
    }
    if self.input_controller.move_backward {
        offset.y -= 1.0
    }
    if self.input_controller.move_left {
        offset.x += 1.0
    }
    if self.input_controller.move_right {
        offset.x -= 1.0
    }

    let graph = &mut engine.scenes[self.scene].graph;

    if let Some(offset) = offset.try_normalize(f32::EPSILON) {
        graph[self.camera]
            .local_transform_mut()
            .offset(offset.scale(0.1));
    }
}
```

If using a custom game loop:

```rs
event_loop.run(move |event, _, control_flow| {
    match event {
        Event::WindowEvent { event, .. } => { /* keyboard */ }
        Event::DeviceEvent { event, .. } => { /* mouse */ }
        _ => (),
    }
}
```

#### "Just pressed" functionality

Properly handling keyboard event requires some trickery, due to input handling not being available in `on_tick()`. Solution:

```rs
pub struct Game {
    input: InputController,
    // ...
}

impl GameState for Game {
    fn on_tick(&mut self, engine: &mut Engine, _dt: f32, _control_flow: &mut ControlFlow) {
        // ...

        match &self.menu_state {
            NumPlayers => {
                if self.input.is_key_just_pressed(Space) { /* ... */ }
            }
        },

        self.input.flush_event_received_state();
    }

    fn on_window_event(&mut self, _engine: &mut Engine, event: WindowEvent) {
        if let WindowEvent::KeyboardInput { input, .. } = event {
            if let Some(key_code) = input.virtual_keycode {
                use ElementState::*;

                match input.state {
                    Pressed => self.input.key_down(key_code),
                    Released => self.input.key_up(key_code),
                }
            }
        }
    }
}

#[derive(Default)]
pub struct InputController {
    // The value is a tuple of previous and last state (true = pressed).
    // Once an entry is added, it's never removed - on key released, the value is set as (false, false).
    //
    key_states: HashMap<VirtualKeyCode, (bool, bool)>,
    // Fyrox doesn't expose input handling APIs in the `on_tick()` function; since in some cases (key
    // kept pressed) it can take several frames to receive the next event, we need a way to understand
    // how to interpret the state in between - we do it through this variable; see `is_key_just_pressed()`.
    //
    event_received: bool,
}

// WATCH OUT!!! It's **crucial** to invoke `flush_event_received_state()` at the end of `on_tick()`,
// otherwise, the "just pressed" functionality won't work.
//
impl InputController {
    pub fn new() -> Self {
        Self::default()
    }

    pub fn flush_event_received_state(&mut self) {
        self.event_received = false
    }

    pub fn key_down(&mut self, key: VirtualKeyCode) {
        self.key_states
            .entry(key)
            .and_modify(|v| *v = (v.1, true))
            // If it wasn't tracked, we assume that it was not pressed before tracking started.
            .or_insert((false, true));

        self.event_received = true;
    }

    pub fn key_up(&mut self, key: VirtualKeyCode) {
        self.key_states
            .entry(key)
            .and_modify(|v| *v = (v.1, false))
            // If it wasn't tracked, we assume that it was pressed before tracking started.
            .or_insert((true, false));

        self.event_received = true;
    }

    pub fn is_key_pressed(&self, key: VirtualKeyCode) -> bool {
        let key_state = self.key_states.get(&key).unwrap_or(&(false, false));
        key_state.1
    }

    // Although is_key_pressed would also do in some cases, e.g. menus, this is still a user-friendly
    // choice, since the alternate API may generate too many keystrokes.
    //
    pub fn is_key_just_pressed(&self, key: VirtualKeyCode) -> bool {
        let key_state = self.key_states.get(&key).unwrap_or(&(false, false));

        let (previously_pressed, currently_pressed) = *key_state;

        // This logic can be compacted, however, it's kept extended for clarity.
        //
        if !currently_pressed {
            false
        } else {
            if !self.event_received {
                // If no events have been received, we assume that the state has not changed; for this
                // reason, "just pressed" is necessarily false.
                //
                false
            } else {
                !previously_pressed && currently_pressed
            }
        }
    }
}
```

## Window configuration

```rs
// Framework doesn't expose the engine, so need to access it on init().
// WATCH OUT! Don't change the title here - see the hello world.

let window = engine.get_window();

let PhysicalSize { width, height } = window.inner_size();

window.set_maximized(true);
window.set_fullscreen(Some(Fullscreen::Borderless(None)));
window.set_inner_size(PhysicalSize {
    width: 1024,
    height: 768,
});

// WATCH OUT! There are currently issues with resizing; this call must be made before set_inner_size(),
// otherwise resizing won't apply.
// See https://github.com/rust-windowing/winit/issues/2306.
//
window.set_resizable(false);
```

## More sophisticated hello world (2d)

```rs
fn init(engine: &mut Engine) -> Self {
  let mut scene = Scene::new();

  /* See #camera */
  let camera = build_camera(&mut scene.graph);

  // Load scene.
  engine.resource_manager.request_model("examples/data/2d/scene.rgs")
    .instantiate_geometry(&mut scene);

  let scene: Handle<Scene> = engine.scenes.add(scene);

  Self { scene, /* ... */ }
}

fn on_tick(&mut self, engine: &mut Engine, _dt: f32, _: &mut ControlFlow) {
    /* Handle input and transform (see #input) */
}
```

## Random

Fyrox uses the `rand` crate, so just add `rand = "*"` as dependency.
