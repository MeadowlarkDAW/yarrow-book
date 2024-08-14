# Adding Style

Now let's make the label more visually interesting.

To be more representative of a real-world application, let us define a struct to hold all of the style related information about our application. This struct has no fields yet, but we will add some later.

For now, let us define a method which loads the styles for the elements we want:

```rust,no_run
# use yarrow::prelude::*;
# 
# #[derive(Clone)]
# pub enum MyAction {}
# 
# #[derive(Default)]
# struct MyApp {
#     main_window_elements: Option<MainWindowElements>,
# }
# 
# impl Application for MyApp {
#     type Action = MyAction;
# 
#     fn on_window_event(
#         &mut self,
#         event: AppWindowEvent,
#         window_id: WindowID,
#         cx: &mut AppContext<MyAction>,
#     ) {
#         match event {
#             AppWindowEvent::WindowOpened => {
#                 if window_id == MAIN_WINDOW {
#                     let mut cx = cx.window_context(MAIN_WINDOW).unwrap();
# 
#                     self.main_window_elements =
#                         Some(MainWindowElements::build(&mut cx));
#                 }
#             }
#             AppWindowEvent::WindowResized => {
#                 if window_id == MAIN_WINDOW {
#                     let mut cx = cx.window_context(MAIN_WINDOW).unwrap();
# 
#                     self.main_window_elements
#                         .as_mut()
#                         .unwrap()
#                         .layout(&mut cx);
#                 }
#             }
#             _ => {}
#         }
#     }
# }
# 
# pub struct MainWindowElements {
#     hello_label: Label,
# }
# 
# impl MainWindowElements {
#     pub fn build(cx: &mut WindowContext<'_, MyAction>) -> Self {
#         Self {
#             hello_label: Label::builder()
#                 .text("Hello World!")
#                 .build(cx),
#         }
#     }
# 
#     pub fn layout(&mut self, cx: &mut WindowContext<'_, MyAction>) {
#         let label_size = self.hello_label.desired_size(cx.res); // 1
# 
#         // Center the label inside the window
#         let window_rect = Rect::from_size(cx.logical_size()); // 2
#         let label_rect = centered_rect(window_rect.center(), label_size); // 3
# 
#         self.hello_label.el.set_rect(label_rect); // 4
#     }
# }
# 
#[derive(Default)]
struct MyStyle {}

impl MyStyle {
    pub fn load(&self, res: &mut ResourceCtx) { // 1
        res.style_system.add( // 2
            ClassID::default(), // 3
            true, // 4
            LabelStyle { // 5
                back_quad: QuadStyle {
                    bg: background_hex(0x641e50),
                    border: border(hex(0xc83ca0), 2.0, radius(10.0)),
                    ..Default::default()
                },
                text_padding: padding_all_same(10.0),
                ..Default::default() // 6
            },
        );
    }
}
# 
# pub fn main() {
#     let (action_sender, action_receiver) = yarrow::action_channel();
# 
#     yarrow::run_blocking(MyApp::default(), action_sender, action_receiver).unwrap();
# }
```

1. Pass in the `ResourceCtx`, which is a context for the globally shared resources in a Yarrow application.
2. Add a style to the context's `StyleSystem`.
3. The "class ID". Only elements that have this class ID will have this style applied to them. The default class ID means to set it as the default style for all elements of this type which don't have a defined class ID.
4. Whether or not this style is a dark theme variant (true) or a light theme variant (false). This allows for easy switching between light and dark variants later.
5. Every element type defines its own custom style struct with custom properties.
6. The `..Default::default()` syntax is handy for not defining properties you do not use.

Now store our new style struct in `MyApp` and load it when the application starts:

```rust,no_run
# use yarrow::prelude::*;
# 
# #[derive(Clone)]
# pub enum MyAction {}
# 
# #[derive(Default)]
struct MyApp {
    main_window_elements: Option<MainWindowElements>,
    style: MyStyle, // new
}

// ...

# impl Application for MyApp {
#     type Action = MyAction;
# 
#     fn on_window_event(
#         &mut self,
#         event: AppWindowEvent,
#         window_id: WindowID,
#         cx: &mut AppContext<MyAction>,
#     ) {
#         match event {
            AppWindowEvent::WindowOpened => {
                if window_id == MAIN_WINDOW {
                    // new
                    self.style.load(&mut cx.res);

                    let mut cx = cx.window_context(MAIN_WINDOW).unwrap();

                    self.main_window_elements =
                        Some(MainWindowElements::build(&mut cx));
                }
            }
#             AppWindowEvent::WindowResized => {
#                 if window_id == MAIN_WINDOW {
#                     let mut cx =
#                         cx.window_context(MAIN_WINDOW).unwrap();
# 
#                     self.main_window_elements
#                         .as_mut()
#                         .unwrap()
#                         .layout(&mut cx);
#                 }
#             }
#             _ => {}
#         }
#     }
# }
# 
# pub struct MainWindowElements {
#     hello_label: Label,
# }
# 
# impl MainWindowElements {
#     pub fn build(cx: &mut WindowContext<'_, MyAction>) -> Self {
#         Self {
#             hello_label: Label::builder()
#                 .text("Hello World!")
#                 .build(cx),
#         }
#     }
# 
#     pub fn layout(&mut self, cx: &mut WindowContext<'_, MyAction>) {
#         let label_size = self.hello_label.desired_size(cx.res);
# 
#         // Center the label inside the window
#         let window_rect = Rect::from_size(cx.logical_size());
#         let label_rect = centered_rect(window_rect.center(), label_size);
# 
#         self.hello_label.el.set_rect(label_rect);
#     }
# }
# 
# #[derive(Default)]
# struct MyStyle {}
# 
# impl MyStyle {
#     pub fn load(&self, res: &mut ResourceCtx) {
#         res.style_system.add(
#             ClassID::default(),
#             true,
#             LabelStyle {
#                 back_quad: QuadStyle {
#                     bg: background_hex(0x641e50),
#                     border: border(hex(0xc83ca0), 2.0, radius(10.0)),
#                 },
#                 text_padding: padding_all_same(10.0),
#                 ..Default::default()
#             },
#         );
#     }
# }
# 
# pub fn main() {
#     let (action_sender, action_receiver) = yarrow::action_channel();
# 
#     yarrow::run_blocking(MyApp::default(), action_sender, action_receiver).unwrap();
# }
```

Now our label is looking fancy!

![Fancy Label](../img/adding_style_1.png)

# Loading a Theme

By default all elements have a style which is very bare-bones (and most of the time colors are set to transparent). If you want a quicker starting place, you can load one of Yarrow's built in themes. A "theme" is simply a function with a few tweakable parameters that adds a bunch of styles.

At the time of this writing, Yarrow has only one built-in theme called "Yarrow dark". To use it, simply add this inside of `MyStyle::load()`:

```rust,no_run
# use yarrow::prelude::*;
# 
# #[derive(Clone)]
# pub enum MyAction {}
# 
# #[derive(Default)]
# struct MyApp {
#     main_window_elements: Option<MainWindowElements>,
#     style: MyStyle,
# }
# 
# impl Application for MyApp {
#     type Action = MyAction;
# 
#     fn on_window_event(
#         &mut self,
#         event: AppWindowEvent,
#         window_id: WindowID,
#         cx: &mut AppContext<MyAction>,
#     ) {
#         match event {
#             AppWindowEvent::WindowOpened => {
#                 if window_id == MAIN_WINDOW {
#                     self.style.load(&mut cx.res);
# 
#                     let mut cx = cx.window_context(MAIN_WINDOW).unwrap();
# 
#                     self.main_window_elements =
#                         Some(MainWindowElements::build(&mut cx));
#                 }
#             }
#             AppWindowEvent::WindowResized => {
#                 if window_id == MAIN_WINDOW {
#                     let mut cx =
#                         cx.window_context(MAIN_WINDOW).unwrap();
# 
#                     self.main_window_elements
#                         .as_mut()
#                         .unwrap()
#                         .layout(&mut cx);
#                 }
#             }
#             _ => {}
#         }
#     }
# }
# 
# pub struct MainWindowElements {
#     hello_label: Label,
# }
# 
# impl MainWindowElements {
#     pub fn build(cx: &mut WindowContext<'_, MyAction>) -> Self {
#         Self {
#             hello_label: Label::builder()
#                 .text("Hello World!")
#                 .build(cx),
#         }
#     }
# 
#     pub fn layout(&mut self, cx: &mut WindowContext<'_, MyAction>) {
#         let label_size = self.hello_label.desired_size(cx.res);
# 
#         // Center the label inside the window
#         let window_rect = Rect::from_size(cx.logical_size());
#         let label_rect = centered_rect(window_rect.center(), label_size);
# 
#         self.hello_label.el.set_rect(label_rect);
#     }
# }
# 
# #[derive(Default)]
# struct MyStyle {}
# 
# impl MyStyle {
#     pub fn load(&self, res: &mut ResourceCtx) {
        yarrow::theme::yarrow_dark::load(Default::default(), res); // new
# 
#         res.style_system.add(
#             ClassID::default(),
#             true,
#             LabelStyle {
#                 back_quad: QuadStyle {
#                     bg: background_hex(0x641e50),
#                     border: border(hex(0xc83ca0), 2.0, radius(10.0)),
#                 },
#                 text_padding: padding_all_same(10.0),
#                 ..Default::default()
#             },
#         );
#     }
# }
# 
# pub fn main() {
#     let (action_sender, action_receiver) = yarrow::action_channel();
# 
#     yarrow::run_blocking(MyApp::default(), action_sender, action_receiver).unwrap();
# }
```

After loading a theme, it's probably a good idea to use a custom class for our custom fancy label so it doesn't conflict with the default one in the theme:

```rust,no_run
# use yarrow::prelude::*;
# 
# #[derive(Clone)]
# pub enum MyAction {}
# 
# #[derive(Default)]
# struct MyApp {
#     main_window_elements: Option<MainWindowElements>,
#     style: MyStyle,
# }
# 
# impl Application for MyApp {
#     type Action = MyAction;
# 
#     fn on_window_event(
#         &mut self,
#         event: AppWindowEvent,
#         window_id: WindowID,
#         cx: &mut AppContext<MyAction>,
#     ) {
#         match event {
#             AppWindowEvent::WindowOpened => {
#                 if window_id == MAIN_WINDOW {
#                     self.style.load(&mut cx.res);
# 
#                     let mut cx = cx.window_context(MAIN_WINDOW).unwrap();
# 
#                     self.main_window_elements = Some(MainWindowElements::build(&mut cx));
#                 }
#             }
#             AppWindowEvent::WindowResized => {
#                 if window_id == MAIN_WINDOW {
#                     let mut cx = cx.window_context(MAIN_WINDOW).unwrap();
# 
#                     self.main_window_elements
#                         .as_mut()
#                         .unwrap()
#                         .layout(&mut cx);
#                 }
#             }
#             _ => {}
#         }
#     }
# }
# 
# pub struct MainWindowElements {
#     hello_label: Label,
# }
# 
# impl MainWindowElements {
#     pub fn build(cx: &mut WindowContext<'_, MyAction>) -> Self {
#         Self {
            hello_label: Label::builder()
                .class(MyStyle::CLASS_FANCY_LABEL) // new
                .text("Hello World!")
                .build(cx),
#         }
#     }
# 
#     pub fn layout(&mut self, cx: &mut WindowContext<'_, MyAction>) {
#         let label_size = self.hello_label.desired_size(cx.res);
# 
#         // Center the label inside the window
#         let window_rect = Rect::from_size(cx.logical_size());
#         let label_rect = centered_rect(window_rect.center(), label_size);
# 
#         self.hello_label.el.set_rect(label_rect);
#     }
# }

// ...

# #[derive(Default)]
# struct MyStyle {}
# 
# impl MyStyle {
#     pub const CLASS_FANCY_LABEL: ClassID = 1;
# 
#     pub fn load(&self, res: &mut ResourceCtx) {
#         yarrow::theme::yarrow_dark::load(Default::default(), res);
# 
        res.style_system.add(
            Self::CLASS_FANCY_LABEL, // changed
            // ...
#             true,
#             LabelStyle {
#                 back_quad: QuadStyle {
#                     bg: background_hex(0x641e50),
#                     border: border(hex(0xc83ca0), 2.0, radius(10.0)),
#                 },
#                 text_padding: padding_all_same(10.0),
#                 ..Default::default()
#             },
        );
#     }
# }
# 
# pub fn main() {
#     let (action_sender, action_receiver) = yarrow::action_channel();
# 
#     yarrow::run_blocking(MyApp::default(), action_sender, action_receiver).unwrap();
# }
```