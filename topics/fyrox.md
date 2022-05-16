# Bevy

- [Bevy](#bevy)
  - [Hello world](#hello-world)
  - [GameState methods](#gamestate-methods)

Notes from the [Fyrox Cheat Book](https://fyrox-book.github.io/fyrox/introduction.html).

## Hello world

```rs
use fyrox::{engine::framework::prelude::*, engine::Engine};

struct Game {}

impl GameState for Game {
    fn init(_engine: &mut Engine) -> Self {
        Self {}
    }
}

fn main() {
    Framework::<Game>::new().unwrap().title("Simple").run();
}
```

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
