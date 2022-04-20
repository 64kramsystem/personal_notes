# Rust FLTK

- [Rust FLTK](#rust-fltk)
  - [General infos](#general-infos)
  - [Sample of basic app](#sample-of-basic-app)
  - [Bugs](#bugs)
  - [Callbacks/events](#callbacksevents)
  - [Multithreading](#multithreading)
  - [Widgets](#widgets)
    - [Hold browser](#hold-browser)
  - [Window icon](#window-icon)
  - [Layouts](#layouts)
    - [Pack](#pack)
  - [FLTK (C++): Sample program, and compilation](#fltk-c-sample-program-and-compilation)

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
window.make_resizable(true);
window.end();
window.show();

app.run().unwrap();
```

## Bugs

Due to a bug in the [underlying libxft library](https://github.com/fltk-rs/fltk-rs/issues/1185), color emoji can't be displayed

## Callbacks/events

Base callbacks and events:

```rs
let mut input = Input::default().with_size(0, 25);
input.set_callback(|input| println!("{}", input.value()));

// Each widget type has their default events; use set_trigger() to change it.
// This triggers a callback for every character change.
//
input.set_trigger(CallbackTrigger::Changed);

// Display a pop-up menu on mouse click on a browser:
//
browser.set_callback(|_| {
    if event_mouse_button() == MouseButton::Right {
        let coords = event_coords();
        menu::MenuItem::new(&["1.\t", "2.\t"]).popup(coords.0, coords.1);
    }
});
```

For events that are not triggered by callbacks, one can use the events handler:

```rs
// It seems that there is no way to trigger an event on Enter keypress for a browser, so we need to
// handle it as global event.
// The typical considerations about key down/up apply; for a tap, FLTK seems not to trigger events
// on key down so fast that multiple events are triggered.
//
browser.handle(|browser, _| {
    // WATCH OUT!! Don't use event_key_down(); if a handler moves the focus to another widget, the
    // handler of the current widget will receive further events, like:
    //
    //     KeyDown
    //     Push | Released | Enter | Leave | Drag | Focus | Unfocus
    //     NoEvent
    //
    // which event_key_down() will match. If one really wants to use that, an approach is to test
    // the focus:
    //
    //     if let Some(focused) = focus() {
    //         if focused.is_same(browser) {
    //
    if event == Event::KeyDown && app::event_key() == Key::Enter {
        if let Some(text) = browser.selected_text() {
            println!("selected: {}", text);
            // Don't forget to return true where event propagation needs to be stopped!
            return true;
        }
    }

    // Propagate the event.
    false
});
```

When a widget needs to interact with another widget on callback, due to ownership, one needs to use either reference counting, or message passing:

```rs
pub enum MessageEvent {
    UpdateList(String),
    // Other events...
}

pub struct PMSpotlightApp {
    searchers_provider: SearchersProvider,
    current_searcher: Option<Box<dyn Searcher>>,
    app: App,
    receiver: Receiver<MessageEvent>,
    browser: HoldBrowser,
    input: Input,
}

impl PMSpotlightApp {
    pub fn build(searchers_provider: SearchersProvider) -> Self {
        let app = App::default();
        let mut window = Window::default()
            .with_size(WINDOW_WIDTH, WINDOW_HEIGHT)
            .with_label(WINDOW_TITLE);
        let pack = Pack::default().size_of(&window);

        // Built-in messaging system. The standard mpsc can also be used.
        //
        let (sender, receiver) = app::channel();

        let mut input = Input::default().with_size(0, 25);
        let mut browser = HoldBrowser::default_fill();

        browser.set_text_size(BROWSER_TEXT_SIZE);
        input.set_trigger(CallbackTrigger::Changed);

        Self::callback_update_list(&mut input, sender.clone());
        // other widgets/callbacks...

        pack.end();
        window.end();
        window.show();

        Self {
            searchers_provider,
            current_searcher: None,
            app,
            receiver,
            browser,
            input,
        }
    }

    pub fn run(&mut self) {
        while self.app.wait() {
            if let Some(event) = self.receiver.recv() {
                match event {
                    UpdateList(pattern) => {
                        self.message_event_update_list(pattern);
                    }
                    // Other events...
                }
            }
        }
    }

    // Implement here callbacks, fltk event handlers and message event handlers.
    // `move` are due to the sender (which is cloned upstream).

    fn callback_update_list(input: &mut Input, sender: Sender<MessageEvent>) {
        input.set_callback(move |input| {
            sender.send(UpdateList(input.value()));
        });
    }

    fn fltk_event_move_from_input_to_list(input: &mut Input, sender: Sender<MessageEvent>) {
        // WATCH OUT!! Only one handle() is supported per widget.
        //
        input.handle(move |input, _| {
            if let Some(focused) = focus() {
                if focused.is_same(input) {
                    if event_key_down(Key::Down) {
                        sender.send(FocusOnList);
                        return true;
                    }
                }
            }
            false
        });
    }

    fn message_event_update_list(&mut self, pattern: String) {
        self.browser.clear();
        // update list here...
    }
}
```

## Multithreading

When using a manual event loop:

```rs
while self.app.wait() { ... }
```

one doesn't need to care about threading issues!

Alternatively, in theory fltk(-rs) is thread-safe: https://github.com/fltk-rs/fltk-rs/issues/31.

## Widgets

### Hold browser

List of (1-) selectable entries.

```rs
let mut browser = browser::HoldBrowser::default().with_size(WINDOW_WIDTH, WINDOW_HEIGHT - 25);
browser.add("First");
browser.add("Second");

if let Some(entry_data) = entry_data {
    self.browser.add_with_data(&entry_text, entry_data);
} else {
    self.browser.add(&entry_text);
}

// WATCH OUT! The entry must already exist!
// `SharedImage` is best used when sending images across the app, since set_icon() does not support
// trait objects, so `Box<dyn ImageExt>` can't be used.
//
let shared_image = SharedImage::from_image(PngImage::from_data(image_bytes).unwrap());
self.browser.set_icon(self.browser.size(), icon);
```

## Window icon

Set this *before* showing the window:

```rs
let win_icon = PngImage::from_data(ICON).unwrap();
win.set_icon(Some(win_icon));
```

Only a few formats are supported; GIF isn't.

## Layouts

See https://fltk-rs.github.io/fltk-book/Layouts.html.

### Pack

```rs
let pack = group::Pack::default().size_of(&window);
input::Input::default().with_size(0, 25);
let mut browser = browser::HoldBrowser::default_fill();
pack.end();
```

## FLTK (C++): Sample program, and compilation

Generate compile line (required `libfltk-dev`), execute it and the output: `eval "$(fltk-config --compile test.cpp)" && ./test`.

Sample program:

```cpp
#include <FL/Fl.H>
#include <FL/Fl_Window.H>
#include <FL/Fl_Hold_Browser.H>

class MyBrowser : public Fl_Hold_Browser {
    void MyCallback2(Fl_Widget *w) {
        void *i = selection();
        if (i) printf("item selected is '%s'..\n", item_text(i));
    }

    static void MyCallback(Fl_Widget *w, void *data) {
        MyBrowser *brow = static_cast<MyBrowser*>(data);
        brow->MyCallback2(w);
    }
public:
    MyBrowser(int X, int Y, int W, int H) : Fl_Hold_Browser(X, Y, W, H) {
        when(FL_WHEN_ENTER_KEY_ALWAYS);
        callback(MyCallback, (void*)this);
    }
};

int main() {
    Fl_Window win(300, 500);
    MyBrowser brow(10, 10, 300-20, 500-20);
    brow.add("aaa"); brow.add("bbb"); brow.add("ccc");
    win.show();
    return Fl::run();
}
```
