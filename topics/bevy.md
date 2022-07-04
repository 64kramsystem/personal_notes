# Bevy

- [Bevy](#bevy)
  - [Setup](#setup)
  - [Hello world](#hello-world)
  - [Architecture concepts](#architecture-concepts)
    - [Components/bundles](#componentsbundles)
    - [Resources](#resources)
    - [Systems](#systems)
      - [Stages](#stages)
    - [States and callbacks](#states-and-callbacks)
      - [System sets and ordering](#system-sets-and-ordering)
      - [Handling state via `iyes_loopless` crate](#handling-state-via-iyes_loopless-crate)
    - [Run criteria](#run-criteria)
      - [System Chaining](#system-chaining)
      - [Events](#events)
    - [Queries/ParamSets](#queriesparamsets)
      - [Change detection](#change-detection)
    - [World](#world)
    - [Commands](#commands)
      - [Recursive entities](#recursive-entities)
    - [Plugins](#plugins)
  - [Rendering/Window](#renderingwindow)
    - [Camera](#camera)
    - [Transforms](#transforms)
    - [Background](#background)
    - [Assets](#assets)
    - [Sprites/rectangles](#spritesrectangles)
      - [Sprite utils](#sprite-utils)
    - [UI/Layout](#uilayout)
      - [Text](#text)
  - [Input handling](#input-handling)
    - [Keyboard](#keyboard)
    - [Mouse](#mouse)
    - [Gamepad](#gamepad)
    - [Touchscreen](#touchscreen)
  - [Other APIs](#other-apis)
    - [Time/Timestep](#timetimestep)
      - [Fixed timestep](#fixed-timestep)
    - [Timers](#timers)
    - [Math/Vecs/Quats etc.](#mathvecsquats-etc)
  - [Utils](#utils)
    - [Running a custom game loop](#running-a-custom-game-loop)
    - [Track Assets Loading](#track-assets-loading)
    - [List All Resource Types](#list-all-resource-types)
    - [Display the framerate](#display-the-framerate)
    - [Exit the application](#exit-the-application)
    - [Useful transforms](#useful-transforms)
  - [General design considerations](#general-design-considerations)
  - [3rd party plugins](#3rd-party-plugins)
    - [`bevy_kira_audio` (audio)](#bevy_kira_audio-audio)
    - [`heron` (easy physics)](#heron-easy-physics)

Notes from the [Bevy Cheat Book](https://bevy-cheatbook.github.io).

## Setup

WATCH OUT!

- Rust edition 2021 is required.
- In order to debug from VSC, add to `launch.json`:
  - `"env": { "LD_LIBRARY_PATH": "${workspaceFolder}/target/debug/deps:${env:HOME}/.rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib" }`

```toml
bevy = {
  version = "0.7.0",
  default-features = true, # Set to false to reduce base plugins
  features = [
    # Default (subset)

    "png", "hdr", "vorbis" # Other: "jpeg", "tga", "bmp", "flac", "wav", "mp3", "zstd"
    "bevy_gilrs",          # Gamepad input support
    "bevy_gltf",

    "rendering",           # Includes many + 2d ("bevy_sprite") + 3d ("bevy_pbr", "bevy_gltf")
                           # ^^ + UI ("bevy_ui", "bevy_text")
    "filesystem_watcher",  # Asset hot-reloading
    "bevy_audio",          # Disable if using "bevy_kira_audio"

    # Optional (subset)

    "dynamic"              # Speedup compilation via dynamic linking (disable for release!) (see VSC note above)
    "serialize",           # `serde` support
  ]
}
```

See [other plugins](https://bevy-cheatbook.github.io/setup/bevy-config.html).

## Hello world

```rust
use bevy::{prelude::*, window::PresentMode};

// Without Component, Query<> will raise an error "the trait bound `Name: bevy::prelude::Component` is not satisfied"
//
#[derive(Component)]
struct Person;

#[derive(Component)]
struct Name(String);

fn add_people(mut commands: Commands) {
    commands
        .spawn()
        .insert(Person)
        .insert(Name("Foo".to_string()));
    commands
        .spawn()
        .insert(Person)
        .insert(Name("Bar".to_string()));
}

struct GreetTimer(Timer);

// Use a timed greeter; without a timer, this is executed continuously (see DefaultPlugins note).
//
fn greet_people_timed(time: Res<Time>, mut timer: ResMut<GreetTimer>, query: Query<&Name, With<Person>>) {
    if timer.0.tick(time.delta()).just_finished() {
        for name in query.iter() {
            println!("Hello {}!", name.0);
        }
    }
}

pub struct HelloPlugin;

impl Plugin for HelloPlugin {
    fn build(&self, app: &mut App) {
        app.insert_resource(GreetTimer(Timer::from_seconds(2.0, true)))
            .add_startup_system(add_people)
            .add_system(greet_people_timed);
    }
}

fn main() {
    App::new()
        // WindowDescriptor stores the window properties.
        //
        .insert_resource(WindowDescriptor {
            title: "Hello W!".to_string(),
            width: 1280.,
            height: 720.,
            present_mode: PresentMode::Immediate, // enable/disable vsync here
            ..Default::default()
        })
        // The DefaultPlugins group includes the basic plugins to run a game; it makes a window popup
        // because it includes WindowPlugin and WinitPlugin. Since it also includes an event loop, the
        // systems will run in an infinite loop.
        //
        .add_plugins(DefaultPlugins)
        // In this plugin, we bundle the main setup.
        //
        .add_plugin(HelloPlugin)
        .run();
}
```

## Architecture concepts

### Components/bundles

Components:

```rust
// ECS design: Don't add Xp as a field to Player - decouple it, so that it can be reused!
// Use wrapper structs for simple data types ("newtype" pattern)
//
#[derive(Component)]
struct PlayerXp(u32);

// Use empty structs to mark entities (and query them)
//
#[derive(Component)]
struct Enemy;
```

Bundles are like templates for entities with common components:

```rust
// WATCH OUT! Don't forget the Bundle attribute, otherwise, if a bundle is accidentally used as component,
// no error is raised.
//
#[derive(Bundle)]
struct PlayerBundle {
  health: Health,
  _p: Player,

  // Bundles can be nested
  #[bundle]
  sprite: SpriteSheetBundle,
}

// Tuples of components are considered bundles
//
(ComponentA, ComponentB, ComponentC)
```

### Resources

Resources can be global or local; try to use globals as little as possible (components make a better design, as they can be shared).

Definition:

```rust
// Any type (struct/enum) can be used.
// Use `Default` attribute for simple ones:
//
#[derive(Default)]
struct StartingLevel(usize);

// More complex types:
//
struct ComplexResource { /* ... fields ... */ }

impl FromWorld for ComplexResource {
    fn from_world(world: &mut World) -> Self {
        // This gives full access to anything in the ECS; e.g. mutating other resources:
        //
        // Infallible methods: `resource[_mut]()`
        // Fallible:           `get_resource[_mut]()`
        //
        let mut x = world.resource_mut::<MyOtherResource>();

        ComplexResource { /* ... */ }
    }
}
```

Instantiation:

```rust
// At app creation (extract):
//
App::new()
    .init_resource::<ComplexResource>() // if it implements `Default` or `FromWorld`
    .insert_resource(StartingLevel(3))  // if not, or if you want to set a specific value

// Use from a system.
// If a resource of the given type exists on insertion, it's overwritten.
//
commands.insert_resource(GoalsReached { main_goal: false, bonus: false });
commands.remove_resource::<MyResource>();
```

Usage:

```rust
// Global (shared).
//
fn complex_system(res: ResMut<ResourceA>) { /* ... */ }

// Local - different systems will access a different instance.
// See [systems](#systems) for starting a system with a preset local.
//
fn my_system1(mut local: Local<MyState>) { /* ... */ }
fn my_system2(mut local: Local<MyState>) { /* ... */ }

// If need to match a resource, use as_deref[_mut]()
//
fn system(key: Option<Res<Key>>) {
  if let Some(key) = key.as_deref() { /* ... */ }
}
```

### Systems

Systems are functions executed by the engine. Access (R/W) to the world from a given system is determined via function parameters.

WATCH OUT!! Don't forget that they run concurrently!

```rust
// Tuples can be used to organize complex parameters:
//
fn complex_system(
    start: Res<StartingLevel>                        // access resource
    (a, mut b): (Res<ResourceA>, ResMut<ResourceB>), // compact access to multiple resources
    mut c: Option<ResMut<ResourceC>>,                // resource that may not exist
) {
  // ...
}
```

Systems are run by adding them when building the app:

```rust
App::new()
    .add_startup_system(init_menu)  // run it only once at launch
    .add_system(move_player)        // run it every frame update
```

Bevy handles resources contention by looking at the system signatures.

A system can run with a preset (preinitialized) resource:

```rs
fn my_system(mut cmd: Commands, config: &MyConfig) { /* ... */ }

// Use #with_system(), depending on the context
//
app.add_system(move |cmd: Commands| { my_system(cmd, &config); })
```

#### Stages

WATCH OUT!! Bevy's states/stages are poorly implemented, and should not be used (together); see the [`iyes_loopless` section](#handling-state-via-iyes_loopless-crate).

Each frame goes through stages; Bevy has 5: `First`, `PreUpdate`, `Update`, `PostUpdate`, `Last`.

By default, user-defined systems are placed in `Update`; if systems are added to the other stages, beware of interactions with Bevy's internal ones.

ECS flushing is performed between stages, so they are currently (Apr/22) the easiest way to flush.

See https://bevy-cheatbook.github.io/programming/stages.html

```rs
// All the derives are required! Strings can also be used, but they're less safe.
//
#[derive(Debug, Clone, PartialEq, Eq, Hash, StageLabel)]
struct DebugStage;

App::new()
    // Add custom stages (`CoreStage` is a Bevy enum.)
    // add_stage_before(), and SystemStage::single_threaded() are also supported.
    //
    .add_stage_after(CoreStage::Update, DebugStage, SystemStage::parallel())
    // Associate systems/sets to stages
    //
    .add_system_to_stage(DebugStage, debug_player_hp)
    .add_system_set_to_stage(DebugState, movement)
```

### States and callbacks

WATCH OUT!! Bevy's states/stages are poorly implemented, and should not be used (together); see the [`iyes_loopless` section](#handling-state-via-iyes_loopless-crate).

States are the global app states. It's convenient to use plugins to group systems to use with states.

Setup:

```rust
#[derive(Debug, Clone, Eq, PartialEq, Hash)]
enum AppState { MainMenu, InGame, Paused }

App::new()
    // add the first app state to the stack
    .add_state(AppState::MainMenu)

    // systems outside of system sets will run regardless of the state
    .add_system(play_music)

    // Different state callbacks /////////////////////////////////////////////////

    // on state active (use this as base)
    .add_system_set(SystemSet::on_update(AppState::MainMenu).with_system(handle_ui_buttons))

    // when entering the state (equivalent of startup systems)
    .add_system_set(SystemSet::on_enter(AppState::MainMenu).with_system(setup_menu))

    // on pause, e.g. things to do when becoming inactive
    .add_system_set(SystemSet::on_pause(AppState::InGame).with_system(hide_enemies))

    // player idle animation while paused
    .add_system_set(SystemSet::on_inactive_update(AppState::InGame).with_system(player_idle))

    // on state both active and paused
    .add_system_set(SystemSet::on_in_stack_update(AppState::InGame).with_system(animate_water))

    // when resuming from pause
    .add_system_set(SystemSet::on_resume(AppState::InGame).with_system(reset_player))

    // on exit (e.g. cleanup)
    .add_system_set(SystemSet::on_exit(AppState::MainMenu).with_system(close_menu))
```

Match:

```rust
fn play_music(app_state: Res<State<AppState>>) {
    match app_state.current() {
        AppState::MainMenu => { /* ... */ }
        AppState::InGame => { /* ... */ }
        AppState::Paused => { /* ... */ }
    }
}
```

Change; all the current state's systems complete before moving to the next system (there's no limit to the number of state changes within a frame):

```rust
// Switch via stack.
// Go into the pause screen, and back into the game.
//
app_state.push(AppState::Paused).unwrap();
app_state.pop().unwrap();

// Direct set.
//
fn enter_game(mut app_state: ResMut<State<AppState>>) {
    // Can fail if we are already in the target state or if another state change is already queued
    app_state.set(AppState::InGame).unwrap();
}
```

WATCH OUT!! Key-initiated state changes require `Input` to be reset:

```rust
fn esc_to_menu(mut keys: ResMut<Input<KeyCode>>, mut app_state: ResMut<State<AppState>>) {
    if keys.just_pressed(KeyCode::Escape) {
        app_state.set(AppState::MainMenu).unwrap();
        keys.reset(KeyCode::Escape);
    }
}
```

#### System sets and ordering

WATCH OUT!! Bevy's states/stages are poorly implemented, and should not be used (together); see the [`iyes_loopless` section](#handling-state-via-iyes_loopless-crate).

A SystemSet is a group of systems. Ordering is accomplished via labels, and `before()`/`after()` APIs.

WATCH OUT!! `after()/before()` don't constrain execution - only ordering. In other words, if a system must run on a certain state only, the system set must still specify the state stage callback (e.g. `on_update()`) - see below.

```rust
// All the derives are required! Strings can also be used, but they're less safe.
//
#[derive(Debug, Clone, PartialEq, Eq, Hash, SystemLabel)]
enum MySystemSet { InputSet, MovementSet }

App::new()
    .add_system_set(
        SystemSet::new()
            .label(InputSet)
            .with_system(keyboard_input)
            .with_system(gamepad_input)
    )
    .add_system_set(
        SystemSet::new()
            .label(MovementSet)
            // SystemSet-level ordering.
            //
            .after(InputSet)
            .with_system(player_movement)
            // System-level labelling and ordering. before()/after() can be called on any system instance.
            //
            .with_system(mysystem1.label("mysys").after(mysystem0)) // via function reference
            .with_system(mysystem2.after("mysys"))                  // via label
    )
    .add_system_set(
        SystemSet::on_update(TurnState::MonsterTurn)
            .label(MonsterThinks)
            .with_system(monster_thinks)
    )
    .add_system_set(
        // Example of user bug! The system set must be constrained: (`on_update(TurnState::MonsterTurn)`),
        // otherwise, it will run on any state.
        SystemSet::new()
            .after(Monster)
            .with_system(monster_moves)
    )
```

Labels and labelled items are M:N, so labels can be shared!

#### Handling state via `iyes_loopless` crate

This crate doesn't have problems with mixing states/stages; it allows proper implementation of states management:

- a Stage is added for each (atomic) system set, so that each works on a committed world view
- ConditionState's are used to manage game state (by filtering system sets)

WATCH OUT! It seems that `before`/`after` can't be added at system level, but only at `ConditionSet` level.

Example of state management:

```rs
// Resource-based conditional function; requires `.run_if(on_rendering_signal)`.
// See the system set declaration below for the non-function-based alternative.
//
// It's discouraged to mutable access unless one is certain of what they're doing; this is a fairly
// simple case, so there's no harm.
//
fn on_rendering_signal(mut commands: Commands, render: Option<Res<DoRender>>) -> bool {
    if render.is_some() {
        commands.remove_resource::<DoRender>();
        true
    } else {
        false
    }
}

// System set that runs on each frame.
app.add_system_set(
    ConditionSet::new()
        .run_if_resource_exists::<DoRender>()      // run_if() must be invoked before with_system().
        .with_system(map_render::map_render)
        .with_system(entity_render::entity_render)
        .into()
);

// System running on a specific state
app.add_system(
    player_input::player_input.run_in_state(AwaitingInput),
);

// SystemSet (`ConditionSet`) running on a specific state
app.add_system_set_to_stage(
    MovePlayer,
    ConditionSet::new()
        .run_in_state(MovePlayer)
        .with_system(movement::movement)
        .into(),
);
```

### Run criteria

Run criteria allow low-level systems running specification. See: https://bevy-cheatbook.github.io/programming/run-criteria.html.

#### System Chaining

Systems can be chained, so that the output of one is the input of the next.

```rust
fn sender() -> u32 { 42 }
fn receiver(In(val): In<u32>) { println!("Value: {}", val) }

// WATCH OUT! 1. Chaining must be registered; 2. The receiver is the second!!
//
app.add_system(sender.chain(receiver))
```

WATCH OUT!

- for passing data around, it's best to use Events;
- coupling systems by chaining them may limit the parallelization;
- if a system requires wide mutable access, it may limit the parallelization.

#### Events

Systems can communicate via a message queue - the `Event`s system. Every reader tracks the events it has read independently, so one can handle the same events from multiple systems.

WATCH OUT!! W/R don't enforce ordering, so if the two W/R systems are not ordered, events may be read on the next frame ("1-frame-lag" problem). Events persists only until the end of the next frame.

It's possible to manually manage events (see [here](https://bevy-cheatbook.github.io/patterns/manual-event-clear.html)).

```rust
struct LevelUpEvent(Entity);

// Generate an event, via EventWriter<T>.
//
fn player_level_up(mut levelup: EventWriter<LevelUpEvent>, query: Query<(Entity, &PlayerXp)>) {
    for (entity, xp) in query.iter() {
        if xp.0 > 1000 { levelup.send(LevelUpEvent(entity)); }
    }
}

// Read an event, via EventReader<T>.
//
fn debug_levelups(mut levelup: EventReader<LevelUpEvent>) {
    for ev in levelup.iter() {
        eprintln!("Entity {:?} leveled up!", ev.0);
    }
}

// Don't forget to register (custom) events!
//
App::new().add_event::<LevelUpEvent>()
```

### Queries/ParamSets

Used to access entity components. WATCH OUT! Bundles must be queried via component(s), not bundle type!

```rust
fn check_zero_health(
    // Find entities that have (`Health` & `Transform`) components, and are optionally a `Player`.
    // Entity is the entity id (not required).
    // With/out<T> are (optional) filters: the entities must include an Ally, but not an Enemy.
    // For an Or filter example, see [Change detection](#change-detection). In order to specify multiple
    // With/out conditions, pass multiple With/out.
    mut query: Query<
        (Entity, &Health, &mut Transform, Option<&Player>),
        (With<Ally>, Without<Enemy>)
    >
) {
    // Iterate all matching entities
    for (eid, health, mut transform, player) in query.iter_mut() {
        if health.hp <= 0.0 { transform.translation = Vec3::ZERO; }
        if let Some(player) = player { /* the current entity is the player! */ }
    }
}


// More complex query param
(With<Collider>, Or<(With<Player>, With<Enemy>)>)

// Components for a specific entity.
//
if let Ok((health, mut transform)) = query.get_mut(entity) {
    // do something with the components
} else {
    // the entity does not have the components from the query
}

// Retrieve an entity that is guaranteed to be unique; panics if not found
//
// APIs: [get_]single[_mut](): combinations of checked (get)/unchecked and im/mutable.
//
fn query_player(mut q: Query<(&Player, &mut Transform)>) {
    if let Ok(player, mut transform) = q.single_mut() { /* ... */ }
}

// Test if an entity has a component
//
pub fn movement(mut events: EventReader<Move>, query: Query<&Player>) {
    for &Move { entity, .. } in events.iter() {
        // Check if the Player has Move.
        //
        if query.get(entity).is_ok() { /* ... */ }
    }
}
```

When it's not possible for systems to query due to mutability conflicts, query sets can be used; they prevent conflicting resources to be accessed at the same time.

```rust
fn reset_health(
  // Allow access to Health in both queries. There is a limit to the queries; it may be 7.
  //
  mut q: ParamSet<(
    Query<&mut Health, With<Enemy>>,
    Query<&mut Health, With<Player>>
  )>,
) {
  for mut health in q.p0_mut().iter_mut() { /* ... */ }
  for mut health in q.p1_mut().iter_mut() { /* ... */ }
}
```

#### Change detection

WATCH OUT 1-frame lag; for `Added`, [stages](#stages) may be required.

For components, use query filters:

- `Added<T>`: component added to an entity, or entity with it created
- `Changed<T>`: some as `Added` or component mutably accessed (`DerefMut` access, not necessarily changed; not query only)

If triggering `Changed` for unchanged variables is undesirable, modify the code upstream to set a variable (`DerefMut`) only if the value changed.

Removal is trickier to detect; see [here](https://bevy-cheatbook.github.io/programming/removal-detection.html).

```rust
// Print the stats of friendly players when they change:
//
fn debug_stats_change(
    query: Query<
        (&Health, &PlayerXp),
        (Without<Enemy>, Or<(Changed<Health>, Changed<PlayerXp>)>),
    >,
) {
    for (health, xp) in query.iter() {
        eprintln!("hp: {}+{}, xp: {}", health.hp, health.extra, xp.0);
    }
}

// Detect new enemies and print their health:
//
fn debug_new_hostiles(query: Query<(Entity, &Health), Added<Enemy>>) {
    for (entity, health) in query.iter() {
        eprintln!("Entity {:?} is now an enemy! HP: {}", entity, health.hp);
    }
}
```

For resources, just query on `Res`:

```rs
fn myfn(res: Res<MyRes>) {
  let is_changed = res.is_changed();
  let is_added = res.is_added();
}
```

### World

It's possible to interact directly with Bevy's storage, `World`, via App#world:

```rs
let world = app.world;

// This is a realworld case where access was outside a system by design; World access is necessary,
// because App doesn't support removing resources.
//
world.remove_resource::<VirtualKeyCode>();

// Queries
//
world.query();          // Doesn't support <With<T>>
world.query_filtered(); // Supports <With<T>>

// Insert a component/bundle; both inserts return EntityMut, which can be conveniently used to separately
// add other components.
// De/spawning flushes the world!
//
world.spawn()
    .insert(foo)
    .insert_bundle((bar, baz))

// Add a component to an entity (usual `[get_]entity[_mut]` format).
//
world.entity_mut(entity)
    .insert(foo)

// Systems with mutable World access are called "exclusive", and block all the other ones; also, accessing
// World can be incompatible with querying some resources.
//
// Exclusive system must be registered with `exclusive_system()`.
//
pub fn excl_movement(world: &mut World) {
  if let Some(player) = world.entity(entity).get_mut::<Player>() { /* ... */ }
}
add_system(excl_movement.exclusive_system())
```

In order to clear entities/resources:

```rs
// Resources can't be easily cleared as of v0.7; see https://github.com/bevyengine/bevy/pull/3212/files
//
world.clear_entities();
```

### Commands

WATCH OUT!! Commands are flushed at the end of a stage; this includes Resources, that when inserted, can't be queried until the next stage! This also applies to entities despawning (which are visible until the end of the stage).

Commands add/remove/update stuff:

```rust
fn spawn_player(mut commands: Commands) {
    // Manage resources
    //
    commands.insert_resource(GoalsReached { main_goal: false, bonus: false });
    commands.remove_resource::<MyResource>();

    // Create a new entity using `spawn`.
    // Insert commands allow chaining additions, which operate on the same entity.
    // WATCH OUT! insert_bundle() adds all the components of a bundle, while insert(Bundle) inserts the
    // bundle as a single component!
    // WATCH OUT! If (the type of) a component exists already in the entity on addition, it's overwritten!
    //
    let entity_id = commands.spawn()
        .insert(ComponentA)                 // add a component
        .insert_bundle(MyBundle::default()) // add a bundle (to the same entity!)
        .id();                              // get the Entity ID

    // Shorthand for creating an entity with a bundle.
    //
    commands.spawn_bundle(PlayerBundle {
        name: PlayerName("Henry".into()),
        _p: Player,
    });

    // Since tuples are bundles, they can be used with spawn_bundle().
    //
    let other = commands.spawn_bundle((
        ComponentA::default(),
        ComponentB::default(),
    )).id();

    // Add/remove components of an existing entity.
    //
    commands.entity(entity_id)
        .insert(ComponentB)
        .remove::<ComponentA>()
        .remove_bundle::<MyBundle>();

    // Despawn an entity.
    //
    commands.entity(other).despawn();
}

fn make_all_players_hostile(mut commands: Commands, query: Query<Entity, With<Player>>) {
    for entity in query.iter() {
        commands.entity(entity).insert(Enemy); // add an `Enemy` component to the entity
    }
}
```

#### Recursive entities

A hierarchy can be established among entities:

```rust
commands.entity(parent).push_children(&[child]);

// Alternative:
//
commands.spawn_bundle(MyParentBundle::default())
    .with_children(|parent| {
        parent.spawn_bundle(MyChildBundle::default());
    });

// Despawn a whole hierarchy:
//
commands.entity(entity).despawn_recursive();
```

Querying; use the `Parent`/`Children` reserved components:

```rust
// Query parent, from children:
//
fn camera_with_parent(q_child: Query<(&Parent, &Transform), With<Camera>>, q_parent: Query<&GlobalTransform>) {
    for (parent, child_transform) in q_child.iter() {
        // `parent` contains the Entity ID we can use to query components from the parent:
        //
        let parent_global_transform = q_parent.get(parent.0);
    }
}

// Query children, from parent:
//
fn process_squad_damage(q_parent: Query<(&MySquadDamage, &Children)>, q_child: Query<&MyUnitHealth>) {
    for (squad_dmg, children) in q_parent.iter() {
        // `children` is a collection of Entity IDs:
        //
        for &child in children.iter() {
          let health = q_child.get(child);
        }
    }
}
```

For transforms, there is special support.

### Plugins

Plugins are convenient to group things to add to the app, e.g. physics, input handling etc.:

```rust
pub struct HelloPlugin;

impl Plugin for HelloPlugin {
    fn build(&self, app: &mut App) {
        app.insert_resource(GreetTimer(Timer::from_seconds(2.0, true)))
          .add_startup_system(add_people)
          .add_system(greet_people_timed);
    }
}

app.add_plugin(Plugin)
```

Plugins can be grouped:

```rust
struct MyPluginGroup;

impl PluginGroup for MyPluginGroup {
    fn build(&mut self, group: &mut PluginGroupBuilder) {
        group
          .add(FooPlugin)
          .add(BarPlugin);
    }
}
```

It's possible to register plugin groups while excluding certain plugins:

```ruby
App::new()
    .add_plugins_with(DefaultPlugins, |plugins| {
        plugins.disable::<LogPlugin>()
    })
```

## Rendering/Window

In order to get the screen size:

```rs
fn spawn_camera(mut commands: Commands, windows: Res<Windows>) {
    let window = windows.get_primary().unwrap();
    camera.transform = Transform::from_xyz(window.width() / 2., -window.height() / 2., 999.);
}
```

### Camera

```rs
// Add the orthographic camera bundle:
//
commands.spawn_bundle(OrthographicCameraBundle::new_2d());

// Query the camera transform:
//
pub fn move_camera(q_camera: Query<&GlobalTransform, With<Camera2d>>) { /* .. */ }
```

### Transforms

Bevy's coordinates are left-hand.

In 2D:

- (0, 0) is at the center of the screen;
- WATCH OUT! The Y axis points upwards; !!! !!! !!! This is very confusing when drawing 2d !!! !!! !!!
- by default, the camera is far (999.9, slightly before Z-clipping), so that the sprites can be displayed, however, in case of custom transforms, this must be take into account;
- it's convenient to put the background on z=0, and other sprites with increasing z (based on priority).

There are two transforms: `Transform` (relative to parent) and `GlobalTransform` (best to use read-only; if a component has no parent, T = GT)

WATCH OUT! GT is synced with T in the `PostUpdate` stage, so be careful.

A transform has three components:

```rs
pub struct Transform {
    /// Position of the entity. In 2d, the last value of the `Vec3` is used for z-ordering.
    pub translation: Vec3,
    /// Rotation of the entity.
    pub rotation: Quat,
    /// Scale of the entity.
    pub scale: Vec3,
}
```

Useful stuff (see also [maths section](#mathvecsquats-etc)):

```rs
Transform::from_xyz(100., 0., 0.)       // from coordinates
Transform::from_scale(Vec3::splat(6.))  // scale

transform.translation.xy()              // Get a 2d vec of the position of a Transform

// Bound transform
let extents = Vec3::from((BOUNDS / 2.0, 0.0));
transform.translation = transform.translation.min(extents).max(-extents);

// Rotate of a given amount
let rotation_delta = Quat::from_rotation_z(rotation_factor * rotation_speed * TIME_STEP);
transform.rotation *= rotation_delta;
```

### Background

Set the background color:

```rs
App::new().insert_resource(ClearColor(Color::rgb(0.4, 0.4, 0.4)))

// In system form
pub fn set_background(mut commands: Commands) {
    commands.insert_resource(BACKGROUND_COLOR); // ClearColor can be set as const
}
```

### Assets

Assets are loaded asynchronously, but the handle is returned (and can be used) immediately.
Asset changes are detected by Bevy, and hot-reloaded; this works fine for simple parts, but more complex ones (e.g. scene files) may require custom reloading logic (via `AssetEvent`).

Load/access assets:

```rust
struct UiFont(Handle<Font>);

struct SpriteSheets {
    map_tiles: Handle<TextureAtlas>,
}

struct ExtraAssets(Vec<HandleUntyped>);

fn load_ui_font(mut commands: Commands, server: Res<AssetServer>) {
  // Multiple `load`s of the same file return the same handle.
  //
  let handle: Handle<Font> = server.load("font.ttf");

  // Store it - avoids collection and consequent unloading.
  //
  commands.insert_resource(UiFont(handle));
}

fn use_sprites(handles: Res<SpriteSheets>, atlases: Res<Assets<TextureAtlas>>, textures: Res<Assets<Texture>>) {
  // The two getters can return None if the assets are not loaded yet.

  if let Some(atlas) = atlases.get(&handles.map_tiles) { /* ... */ }
  if let Some(map_tex) = textures.get("map.png") { /* ... */ }
}

fn paths_and_clones(mut commands: Commands, server: Res<AssetServer>) {
  // Load with "Asset path".
  //
  let mesh: Handle<Mesh> = server.load("scene.gltf#Mesh0/Primitive0");

  // Strong and weak clones (have standard strong/weak referencing behavior).
  //
  let strong_h_clone = mesh.clone();
  let weak_h_clone = mesh.clone_weak();

  // Untyped clones allow storing mixed-typed collections.
  //
  let untyped_clone = my_collection.clone_untyped();

  // Untyped loading! Formats are detected by extension.
  //
  if let Ok(handles) = server.load_folder("extra") {
    commands.insert_resource(ExtraAssets(handles));
  }
}
```

Manual assets generation (e.g. for procedural generation):

```rust
fn add_material(mut materials: ResMut<Assets<StandardMaterial>>) {
  let new_mat = StandardMaterial { base_color: Color::rgba(0.25, 0.50, 0.75, 1.0), unlit: true, ..Default::default() };
  materials.add(new_mat);
}
```

For custom asset events/hooks, see https://bevy-cheatbook.github.io/features/assets.html#assetevent.

### Sprites/rectangles

For textures/images, Y points downwards. This is consistent with the majority of the image formats, but inconsistent with the rest of Bevy (and OpenGL).

Add a sprite:

```rs
commands
    .spawn_bundle(SpriteBundle {
        texture: asset_server.load("branding/icon.png"),

        // If we want a plain rectangle, use a flat Color instead of a texture.
        // We can also use `custom_size` not to use a texture's dimensions.
        //
        // sprite: Sprite {
        //     color: Color::rgb(0.25, 0.25, 0.75),
        //     custom_size: Some(Vec2::new(50.0, 50.0)),
        //     ..default()
        // },

        // The transform is a queryable component!
        transform: Transform::from_xyz(100., 0., 0.),

        // optional; use flip_y to invert the Y axis direction.
        sprite: Sprite { flip_y: true, flip_x: false, ..Default::default() },

        ..default()
    })
    // Add another component, so we can query this sprite; for example, via Query<(&Transform, &OtherComponent)>
    .insert(OtherComponent);
```

Sprite sheets; they're indended to represent multiple states of a sprite, rather than heterogeneous sprites:

```rs
fn setup(
    mut commands: Commands,
    asset_server: Res<AssetServer>,
    mut texture_atlases: ResMut<Assets<TextureAtlas>>,
) {
    let texture_atlas: TextureAtlas = TextureAtlas::from_grid(
        asset_server.load("textures/rpg/chars/gabe/gabe-idle-run.png"),
        Vec2::new(24.0, 24.0), 7, 1, // sprite size, columns, rows
    );

    let texture_atlas_handle: Handle<TextureAtlas> = texture_atlases.add(texture_atlas);

    let mut sprite_sheet_bundle = SpriteSheetBundle {
        texture_atlas: texture_atlas_handle,
        transform: Transform::from_scale(Vec3::splat(6.0)), // optional: scale sprites
        ..default()
    };

    commands
        .spawn_bundle(sprite_sheet_bundle)
        .insert(AnimationTimer(Timer::from_seconds(0.1, true)));
}
```

If one wants to use individual textures from an atlas (via index):

```rs
// Either specify the index while instantiating the SpriteSheetBundle:
//
SpriteSheetBundle {
    sprite: TextureAtlasSprite { index: 1, ..default() },
    ...
}

// Or set it separately:
//
sprite_sheet_bundle.sprite.index = 1;
```

#### Sprite utils

Bevy provides a collision check API:

```rs
sprite::collide_aabb::collide(a_pos: Vec3, a_size: Vec2, b_pos: Vec3, b_size: Vec2) -> Option<Collision>;
```

### UI/Layout

In order to use a UI, don't forget to spawn a UI camera (`UiCameraBundle`).

Since the Y starts at the bottom, the UI flows from there; use `FlexDirection::ColumnReverse` to flow from the top.

#### Text

Creating text:

```rs
let text = Text {
    sections: vec![
        TextSection {
            value: "Score: ".to_string(), // "\n" can be used as newline
            style: TextStyle {
                font: asset_server.load("fonts/FiraSans-Bold.ttf"),
                font_size: 40.0,
                color: Color::rgb(0.5, 0.5, 1.0),
            },
        },
        TextSection {
            value: "".to_string(),
            style: TextStyle {
                font: asset_server.load("fonts/FiraSans-Bold.ttf"),
                font_size: 40.0,
                color: Color::rgb(1.0, 0.5, 0.5),
            },
        },
    ],
    ..Default::default()
};
let style = Style {
    position_type: PositionType::Absolute,
    position: Rect {
        top: SCORE_PADDING,
        left: SCORE_PADDING,
        ..Default::default()
    },
    ..Default::default()
};

let text_bundle = TextBundle { text, style, ..Default::default() };
let scoreboard = ScoreBoard { score: 0 }; // assumed to be a component

commands.spawn_bundle(text_bundle)
        .insert(scoreboard);
```

Update it:

```rs
// A more sophisticated approach is to use change detection.
//
fn update_scoreboard(scoreboard: Res<Scoreboard>, mut query: Query<(&mut Text, &Scoreboard)>) {
    let (mut text, scoreboard) = query.single_mut();
    text.sections[1].value = format!("{}", scoreboard.score);
}
```

## Input handling

Input handling can be managed via resources (higher level) and events (lower level).

### Keyboard

Control is `LControl`/`RControl`.

```rust
fn keyboard_input(keys: Res<Input<KeyCode>>) {
  if keys.just_pressed(KeyCode::Space) { /* Space was just pressed */ }
  if keys.just_released(KeyCode::LControl) { /* Left Ctrl was released */ }
  if keys.pressed(KeyCode::W) { /* W is (being) held down */ }

  if keys.any_pressed([LControl, RControl]) { }
}

fn keyboard_events(mut key_evr: EventReader<KeyboardInput>) {
  use bevy::input::ElementState;

  for key: &KeyCode in keys.get_just_pressed() { }

  // Scan codes are provided, but currently not usable.
  //
  for ev in key_evr.iter() {
    match ev.state {
      ElementState::Pressed => println!(ev.key_code),
      ElementState::Released => println!(ev.key_code)
    }
  }
}
```

### Mouse

Mouse center is bottom left (!).

Grab the mouse: see https://bevy-cheatbook.github.io/window/mouse-grab.html.

Motion:

```rs
fn cursor_position(windows: Res<Windows>) {
    let window = windows.get_primary().unwrap();

    if let Some(position) = window.cursor_position() {
        // cursor is inside the window, position given
    }
}
```

Buttons/wheel:

```rs
fn mouse_button_input(buttons: Res<Input<MouseButton>>) {
  if buttons.just_pressed(MouseButton::Left) { /* Left button was pressed */ }
  if buttons.just_released(MouseButton::Left) { /* Left Button was released */ }
  if buttons.pressed(MouseButton::Right) { /* Right Button is being held down */ }
}

fn mouse_button_events(mut mousebtn_evr: EventReader<MouseButtonInput>) {
  use bevy::input::ElementState;

  for ev in mousebtn_evr.iter() {
    match ev.state {
      ElementState::Pressed => println!(ev.button),
      ElementState::Released => println!(ev.button),
    }
  }
}

fn scroll_events(mut scroll_evr: EventReader<MouseWheel>) {
  use bevy::input::mouse::MouseScrollUnit;

  for ev in scroll_evr.iter() {
    match ev.unit {
      MouseScrollUnit::Line => println!("Scroll (line units): vertical: {}, horizontal: {}", ev.y, ev.x),
      MouseScrollUnit::Pixel => println!("Scroll (pixel units): vertical: {}, horizontal: {}", ev.y, ev.x),
    }
  }
}
```

Relative motion:

```rust
fn mouse_motion(mut motion_evr: EventReader<MouseMotion>) {
  for ev in motion_evr.iter() {
    println!("Mouse moved: X: {} px, Y: {} px", ev.delta.x, ev.delta.y);
  }
}
```

Cursor:

```rust
fn cursor_position(windows: Res<Windows>) {
  // Games typically only have one window (the primary window).
  // For multi-window applications, you need to use a specific window ID here.
  let window = windows.get_primary().unwrap();

  if let Some(_position) = window.cursor_position() {
    // cursor is inside the window, position given
  } else {
    // cursor is not inside the window
  }
}

// To detect when the pointer is moved, use CursorMoved events to get the updated coordinates:

fn cursor_events(mut cursor_evr: EventReader<CursorMoved>) {
  for ev in cursor_evr.iter() {
    println!("New cursor position: X: {}, Y: {}, in Window ID: {:?}", ev.position.x, ev.position.y, ev.id);
  }
}
```

### Gamepad

Backed by the [gilrs library](https://gitlab.com/gilrs-project/gilrs).

Connection:

```rust
// Simple resource to store the ID of the connected gamepad; we need to know which gamepad to use for player input.
//
struct MyGamepad(Gamepad);

fn gamepad_connections(mut commands: Commands, my_gamepad: Option<Res<MyGamepad>>, mut gamepad_evr: EventReader<GamepadEvent>) {
  for GamepadEvent(id, kind) in gamepad_evr.iter() {
    match kind {
      GamepadEventType::Connected => {
        // If we don't have any gamepad yet, use this one
        if my_gamepad.is_none() { commands.insert_resource(MyGamepad(*id)) }
      }
      GamepadEventType::Disconnected => {
        // If it's the one we previously associated with the player, disassociate it:
        if let Some(MyGamepad(old_id)) = my_gamepad.as_deref() {
          if old_id == id { commands.remove_resource::<MyGamepad>() }
        }
      }
      // other events are irrelevant
      _ => {}
    }
  }
}
```

Input:

```rust
fn gamepad_input(axes: Res<Axis<GamepadAxis>>, buttons: Res<Input<GamepadButton>>, my_gamepad: Option<Res<MyGamepad>>) {
  let gamepad_id = if let Some(gp) = my_gamepad { gp.0 } else { return };

  // The joysticks are represented using a separate axis for X and Y

  let axis_lx = GamepadAxis(gamepad_id, GamepadAxisType::LeftStickX);
  let axis_ly = GamepadAxis(gamepad_id, GamepadAxisType::LeftStickY);

  if let (Some(x), Some(y)) = (axes.get(axis_lx), axes.get(axis_ly)) {
    let left_stick_pos = Vec2::new(x, y); // for convenience

    // implement a dead-zone to ignore small inputs
    if left_stick_pos.length() > 0.1 { /* ... */ }
  }

  let jump_button = GamepadButton(gamepad_id, GamepadButtonType::South);
  let heal_button = GamepadButton(gamepad_id, GamepadButtonType::East);

  if buttons.just_pressed(jump_button) { /* button pressed: make the player jump */ }
  if buttons.pressed(heal_button) { /* button being held down: heal the player */ }
}
```

### Touchscreen

See https://bevy-cheatbook.github.io/features/input-handling.html#touchscreen.

## Other APIs

### Time/Timestep

Don't use `std::time::Instant::now()` to get the time - use `Res<Time>`: due to parallel execution, the former will be inconsistent/inaccurate (respect to the game time).

In order to run with variable timestep:

```rs
pub fn paddle_input(time: Res<Time>, ...) {
    velocity.x = direction * speed * time.delta();
}
```

#### Fixed timestep

Use to run systems at fixed interval.

```rust
use bevy::core::FixedTimestep;

App::new()
    .add_system_set(
        SystemSet::new()
            .with_run_criteria(FixedTimestep::step(1.0)) // 1 Hz
            .with_system(system_1)
    )
    .add_system_set(
        SystemSet::new()
            .with_run_criteria(FixedTimestep::step(0.5)) // 2 Hz
            .with_system(system_2)
    )
```
WATCH OUT!

- events persist 2 frames, so if the fixed timestep is too slow, it will miss them;
- timing is not exact, due to the inexact run criteria underlying nature;
- since states are internally implemented via run criteria, they will conflict with other run criteria uses, e.g. fixed timestep.
- etc.etc.!.

See https://bevy-cheatbook.github.io/features/fixed-timestep.html (and referred examples) for more details.

### Timers

```rs
app.insert_resource(EnemyBulletTimer(Timer::new(
    Duration::from_secs_f32(ENEMY_BULLET_INTERVAL), false, // (duration, repeating)
)))

pub fn spawn_enemy_bullets(
    mut enemy_bullet_timer: ResMut<EnemyBulletTimer>,
    time: Res<Time>,
) {
    enemy_bullet_timer.0.tick(time.delta());

    if enemy_bullet_timer.0.finished() {
        // ...
        enemy_bullet_timer.0.reset();
    }
}
```

### Math/Vecs/Quats etc.

`Vec`a and `Quat`s are from the `glam` crate.

```rust
// - {N}: size (2,3,...)
// - {A}: axis (X, Y, Z)

const_vecN!([x., y.])        // Create a const Vec2; also for: Mat2/3/4, Quat, Vec3/3a/4
vec3.truncate()              // Convert a Vec3 to Vec2, discarding z
vec.normalize()              // WATCH OUT! Returns NaN when the vector is zero
vec.normalize_or_zero()      // Convenient - returns a zero vector when zero
Vec{N}::splat(val)           // Create a Vec with all the components set to val
Vec3::from((to_player, z))   // Convert Vec2+z to Vec3
vec2.extend(z)               // Convert Vec2+z to Vec3

// ??? Not in Glam from Bevy 0.7; just use [angle.cos(), angle.sin()]
Vec2::from_rotation(rads)    // Convert angle to Vec2

Vec{N}::{A}                  // Unit vector in the given axis

// Gets the minimal rotation for transforming `from` to `to`; the rotation is in the plane spanned by the two vectors.
//
Quat::from_rotation_arc(from3, to3)

// Get a rotation from the angle (in radians) around the given axis
//
Quat::from_rotation_{A}(angle)
```

## Utils

### Running a custom game loop

```rs
#[derive(Component)]
struct Number(u32);

fn add_number(mut commands: Commands) {
    commands.spawn().insert(Number(42));
}

fn print_number(query: Query<&Number>) {
    println!("Number: {}!", query.single().0);
}

fn manual_runner(mut app: App) {
    loop {
        app.update();
        sleep(Duration::from_secs(1));
    }
}

fn main() {
    App::new()
        .add_startup_system(add_number)
        .add_system(print_number)
        .set_runner(manual_runner)
        .run();
}
```

Shorter version:

```rs
let mut app = App::new();
app.add_startup_system(add_number).add_system(print_number);

loop {
    app.update();
    sleep(Duration::from_secs(1));
}
```

### Track Assets Loading

```rust
#[derive(Default)]
struct AssetsLoading(Vec<HandleId>);

fn setup(server: Res<AssetServer>, mut loading: ResMut<AssetsLoading>) {
    let font: Handle<Font> = server.load("my_font.ttf");
    let menu_bg: Handle<Texture> = server.load("menu.png");
    loading.0.push(font.id);
    loading.0.push(menu_bg.id);
}

fn check_assets_ready(server: Res<AssetServer>, loading: Res<AssetsLoading>) {
    use bevy::asset::LoadState;

    match server.get_group_load_state(loading.0.iter().copied()) {
        LoadState::Failed => { /* one of our assets had an error */ }
        LoadState::Loaded => {}
        _                 => { return; /* NotLoaded/Loading: not fully ready yet */ }
    }

    // all assets are now ready
}
```

### List All Resource Types

List all the types that have been added (by the dev, not by Bevy) as resources:

```rust
fn print_resources(archetypes: &Archetypes, components: &Components) {
    // `get_short_name()` removes the path information, e.g `bevy_audio::audio::Audio` -> `Audio`;
    // in order to see the path, just use `info.name()`.
    let mut resources = archetypes
        .resource()
        .components()
        .map(|id| components.get_info(id).unwrap())
        .map(|info| TypeRegistration::get_short_name(info.name()))
        .collect::<Vec<_>>();

    resources.sort();
    resources.iter().for_each(|name| println!("{}", name));
}
```

### Display the framerate

```rust
use bevy::diagnostic::{FrameTimeDiagnosticsPlugin, LogDiagnosticsPlugin};

App::new()
    .add_plugin(LogDiagnosticsPlugin::default())
    .add_plugin(FrameTimeDiagnosticsPlugin::default())
```

### Exit the application

```rust
use bevy::app::AppExit;

fn exit_system(mut exit: EventWriter<AppExit>) {
    exit.send(AppExit);
}

// For prototyping, Bevy provides a system, to exit on pressing the Esc key:
//
App::new().add_system(bevy::input::system::exit_on_esc_system)
```

### Useful transforms

- Convert cursor to world coordinates: see https://bevy-cheatbook.github.io/cookbook/cursor2world.html#convert-cursor-to-world-coordinates.
- Custom camera projection: see https://bevy-cheatbook.github.io/cookbook/custom-projection.html.
- Pan+Orbit camera: see https://bevy-cheatbook.github.io/cookbook/pan-orbit-camera.html.

## General design considerations

- It's easier to understand how entities are composed, by defining and spawining bundles, rather than inserting components individually (`.spawn().insert(...)`).

## 3rd party plugins

- `bevy_inspector_egui`: display a window with the desired entities and components
- `bevy_ecs_tilemap`: for conveniently/efficiently handling tilemaps
- `bevy_ecs_ldtk`: specialized `bevy_ecs_tilemap` for LDTK
- `leafwing_input_manager`: more advanced input
- `bevy_asset_loader`: convenient synchronous assets loading
- `benimator`: spritesheets
- `bevy_rapier`: physics (lower level, via Rapier)

### `bevy_kira_audio` (audio)

It's advised to use the `bevy_kira_audio` crate for audio. WATCH OUT!

- must specify the Bevy features, and exclude `bevy_audio` and `vorbis`;
- a single audio channel can play multiple sounds;
- it seems that audio channels are intended to be permanent, since there are no removal APIs;
- v0.10 has different APIs from v0.9.

```rs
// For convenience, a struct with all the sound asset handles can be created, but it's not strictly
// necessary, as Bevy caches the assets.

use bevy_kira_audio::{Audio, AudioApp, AudioChannel, AudioPlugin, AudioSource};

// Define one type for each extra channel. The `volume` field is optional; it's a convenience to keep
// track of the volume, since it can't be queried; it's not needed if only playing at max volume.
//
struct Channel1State {
    volume: f32,
}

app.add_plugin(AudioPlugin)
   .add_audio_channel::<Channel1State>()
   .add_startup_system(prepare_audio)
   ...

fn prepare_audio(mut commands: Commands, asset_server: Res<AssetServer>) {
    // Required if we use a (the) channel volume resource.
    commands.insert_resource(Channel1State { volume: 1.0 });
}

// There is a main channel, by default - Res<Audio>
//
fn play_main_channel(asset_server: Res<AssetServer>, audio: Res<Audio>) {
    let source_h = asset_server.load("sounds/01.ogg");

    // Playback-related methods

    audio.play_looped(source_h);
    // audio.play(source_h);
    audio.stop();
    audio.pause();
    audio.resume();
}

// Non-main channels are queried via `Res<AudioChannel<T>>`.
//
fn play_channel_1(
    sounds: Res<Sounds>,
    channel1: Res<AudioChannel<Channel1State>>,
    mut channel_1_state: ResMut<Channel1State>,
) {
    // Set (and store) the volume.

    let source_h = asset_server.load("sounds/01.ogg");
    channel_1_state.volume = 0.7;
    channel1.set_volume(channel_1_state.volume);
    channel1.play(source_h);
}
```

### `heron` (easy physics)

Hello world, with ball that falls on the ground:

```rs
app.add_plugin(PhysicsPlugin::default())
   insert_resource(Gravity::from(Vec2::new(0.0, -600.0))) // Define the gravity (Y points up!)

// Bundles must contain at least a `GlobalTransform`

// Ground

let size = Vec2::new(1000.0, 50.0);
commands
    .spawn_bundle(SpriteBundle {
        sprite: Sprite { color: Color::WHITE, custom_size: Some(size), ..Default::default()},
        transform: Transform::from_translation(Vec3::new(0.0, -300.0, 0.0)),
        ..Default::default()
    })

    // Make it a rigid body
    .insert(RigidBody::Static)

    // Attach a collision shape
    .insert(CollisionShape::Cuboid { half_extends: size.extend(0.0) / 2.0, border_radius: None })

    // Define restitution (so that it bounces)
    .insert(PhysicMaterial { restitution: 0.5, ..Default::default() });

// Ball

let size = Vec2::new(30.0, 30.0);
commands
    .spawn_bundle(SpriteBundle {
        sprite: Sprite { color: Color::GREEN, custom_size: Some(size), ..Default::default() },
        transform: Transform::from_translation(Vec3::new(-400.0, 200.0, 0.0)),
        ..Default::default()
    })
    .insert(RigidBody::Dynamic)
    .insert(CollisionShape::Cuboid { half_extends: size.extend(0.0) / 2.0, border_radius: None, })
    .insert(PhysicMaterial { restitution: 0.7, ..Default::default() });

    // Add an initial velocity. (it is also possible to read/mutate this component later).
    // Also rotates.
    .insert(Velocity::from(Vec2::X * 300.0).with_angular(AxisAngle::new(Vec3::Z, -PI)))
```
