# Your First Element

Let's start by adding a `Label` element with the text "Hello World!".

## A Place to Store our Elements

First we will need to create a struct to hold our elements for the main window. Add the following struct, and add it as a field in `MyApp`:

```rust,no_run
# use yarrow::prelude::*;
# 
# #[derive(Clone)]
# pub enum MyAction {}
# 
#[derive(Default)]
struct MyApp {
    main_window_elements: Option<MainWindowElements>, // new
}
#
# impl Application for MyApp {
#     type Action = MyAction;
# }

pub struct MainWindowElements {
    hello_label: Label,
}
#
# pub fn main() {
#     let (action_sender, action_receiver) = yarrow::action_channel();
# 
#     yarrow::run_blocking(
#         MyApp::default(),
#         action_sender,
#         action_receiver,
#     )
#     .unwrap();
# }
```

So why can't we just add `hello_label` directly to our `MyApp` struct? The reason is that Yarrow applications are designed to work even when the main window isn't open. Not only is this a useful concept for audio plugins, but this allows the same behavior to work for both the main window and any child windows in our application (Yarrow has first-class multi-window support!)

## The Build Function

Now we must define a function to "build" our elements when the window opens. To do this, add the following method to `MainWindowElements`, and call this build function from the `on_window_event` trait method:

```rust,no_run
# use yarrow::prelude::*;
# 
# #[derive(Clone)]
# pub enum MyAction {}
# 
# #[derive(Default)]
# struct MyApp {
#     main_window_elements: Option<MainWindowElements>, // new
# }
#
impl Application for MyApp {
    type Action = MyAction;

    // new
    fn on_window_event(
        &mut self,
        event: AppWindowEvent,
        window_id: WindowID,
        cx: &mut AppContext<MyAction>,
    ) {
        match event {
            AppWindowEvent::WindowOpened => {
                if window_id == MAIN_WINDOW { // 1
                    let mut cx = cx.window_context(MAIN_WINDOW).unwrap(); // 2

                    self.main_window_elements =
                        Some(MainWindowElements::build(&mut cx)); // 3
                }
            }
            _ => {}
        }
    }
}
# 
# pub struct MainWindowElements {
#     hello_label: Label,
# }

impl MainWindowElements {
    pub fn build(cx: &mut WindowContext<'_, MyAction>) -> Self {
        todo!()
    }
}
#
# pub fn main() {
#     let (action_sender, action_receiver) = yarrow::action_channel();
# 
#     yarrow::run_blocking(
#         MyApp::default(),
#         action_sender,
#         action_receiver,
#     )
#     .unwrap();
# }
```

1. Check the ID of the window to see if it is the main window. This will always be the case for our simple app, but this won't be the case if we add more windows in the future.
2. Get the context for the main window. This context is what we add elements to.
3. Build the main window elements and store it in our application struct.

## Building the Label Element

Now we finally get to build the label! Inside `MainWindowElements::build`, add the following:

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
    pub fn build(cx: &mut WindowContext<'_, MyAction>) -> Self {
        // changed
        Self {
            hello_label: Label::builder() // 1
                .text("Hello World!") // 2
                .build(cx), // 3
        }
    }
# }
#
# pub fn main() {
#     let (action_sender, action_receiver) = yarrow::action_channel();
# 
#     yarrow::run_blocking(MyApp::default(), action_sender, action_receiver).unwrap();
# }
```

1. All included elements in Yarrow use the [builder pattern](https://rust-unofficial.github.io/patterns/patterns/creational/builder.html).
2. Elements can include any custom property in their builders. In this case we use the `text` property to set the text of the label.
3. Finally, finish building the element by adding it to the window context.
