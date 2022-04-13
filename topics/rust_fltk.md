# Rust FLTK

- [Rust FLTK](#rust-fltk)
  - [General infos](#general-infos)
  - [Sample of basic app](#sample-of-basic-app)
  - [Callbacks](#callbacks)
  - [Events](#events)
  - [Message passing](#message-passing)
  - [Widgets](#widgets)
    - [Hold browser](#hold-browser)
  - [Layouts](#layouts)
    - [Pack](#pack)

## General infos

Widgets have a `value()` method, that returns the current value.

## Sample of basic app

```rs
let app = App::default();
let mut window = Window::default().with_size(WINDOW_WIDTH, WINDOW_HEIGHT);
let pack = Pack::default().size_of(&window);

let mut input = input::Input::default().with_size(0, 25);

let mut browser = HoldBrowser::default_fill();

pack.end();
window.end();
window.show();

app.run().unwrap();
```

## Callbacks

```rs
let mut frame = Frame::default().with_size(200, 100).center_of(&wind);
let mut button = Button::new(160, 210, 80, 40, "Click me!");

button.set_callback(move |_| frame.set_label("Hello world"));
```

## Events

```rs
// Each widget type has their default events; use set_trigger() to change it.
//
input.set_trigger(enums::CallbackTrigger::Changed);

input.set_callback(|input| println!("{}", input.value()));

browser.set_callback(|_| {
    if app::event_mouse_button() == app::MouseButton::Right {
        let coords = app::event_coords();
        menu::MenuItem::new(&["1.\t", "2.\t"]).popup(coords.0, coords.1);
    }
});

// It seems that there is no way to trigger an event on Enter keypress for a browser, so we need to
// handle it as global event.
// Don't forget to check the focus, otherwise, multiple events will be sent (one for each widget, likely).
// Don't forget to return true where event propagation needs to be stopped!
// The typical considerations about key down/up apply.
//
browser.handle(|browser, _| {
    if let Some(focused) = focus() {
        if focused.is_same(browser) {
            // An alternative check is `event.intersects(Event::KeyDown) && event_key() == Key::Enter`
            //
            if event_key_down(Key::Enter)
                match browser.selected_text() {
                    Some(text) => {
                        println!("selected: {}", text);
                        return true; // stop event propagation
                    }
                    None => {
                        println!("auto: {}", ENTRIES[0]);
                        return true;
                    }
                }
            }
        }
    }

    false // propagate the event
});
```

## Message passing

There are different ways; this is one:

```rs
let (sender_i, receiver) = app::channel();

input.set_callback(move |input| {
    sender_i.send(PatternUpdate(input.value()));
});

// Use manual event loop instead of app.run().
//
while app.wait() {
    if let Some(event) = receiver.recv() {
      match event {
        // ...
      }
    }
}
```

## Widgets

### Hold browser

List of (1-) selectable entries.

```rs
let mut browser = browser::HoldBrowser::default().with_size(WINDOW_WIDTH, WINDOW_HEIGHT - 25);
browser.add("First");
browser.add("Second");
```

## Layouts

See https://fltk-rs.github.io/fltk-book/Layouts.html.

### Pack

```rs
let pack = group::Pack::default().size_of(&window);
input::Input::default().with_size(0, 25);
let mut browser = browser::HoldBrowser::default_fill();
pack.end();
```
