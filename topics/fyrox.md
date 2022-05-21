# Fyrox

- [Fyrox](#fyrox)
  - [Hello world](#hello-world)
  - [Misc notes and concepts](#misc-notes-and-concepts)
  - [GameState methods](#gamestate-methods)
  - [Pool](#pool)
  - [Scene](#scene)
    - [Concepts/Terminology](#conceptsterminology)
    - [Graph](#graph)
    - [Sprites (2d)](#sprites-2d)
    - [Camera](#camera)
  - [Transforms](#transforms)
  - [Events](#events)
    - [Window events](#window-events)
    - [Input](#input)
  - [Window configuration](#window-configuration)
  - [More sophisticated hello world (2d)](#more-sophisticated-hello-world-2d)

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
// Reservation; used to extract an object and put it back. Reserved objects can't be borrowed in the meanwhile.
//
let (ticket, mut obj): Ticket<T>, T = pool.take_reserve(handle) // Format: [try_]take_reserve()
// ...mess with obj...
pool.put_back(ticket, obj)
pool.forget_ticket(ticket)                                      // Put back

// Get a handle from a ref/
//
let str_handle = pool.handle_of(str_ref);
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

Other operations:

```rs
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

This makes it easier to create nodes for 2d games. Use it as root node, and attach/detach sprite nodes are required; the only transform they need is a local scale to the sprite size.

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

Using the default game loop:

```rs
// Typical type.
//
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
            match key_code {
                VirtualKeyCode::W => {
                    if input.state == ElementState::Pressed {
                        println!("W");
                    }
                }
                VirtualKeyCode::S => {
                    if input.state == ElementState::Pressed {
                        println!("S");
                    }
                }
                VirtualKeyCode::A => {
                    if input.state == ElementState::Pressed {
                        println!("A");
                    }
                }
                VirtualKeyCode::D => {
                    if input.state == ElementState::Pressed {
                        println!("D");
                    }
                }
                _ => (),
            }
        }
    }
}

fn on_device_event(&mut self, _engine: &mut Engine, _device_id: DeviceId, event: DeviceEvent) {
    if let DeviceEvent::MouseMotion { delta } = event {
        println!("Mouse: {:?}", delta);

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

// WATCH OUT! This prevents also programmatic changes.
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
