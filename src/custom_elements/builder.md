# Custom Elements: Builder

Elements in yarrow should use the [`builder pattern`](https://rust-unofficial.github.io/patterns/patterns/creational/builder.html) in Rust.

Let's start by defining the builder struct:

```rust
pub struct MyCustomElementBuilder<A: Clone + 'static> { // 1
    pub on_selected_action: Option<A>, // 2
    pub disabled: bool, // 3

    // 4
    pub class: Option<ClassID>,
    pub z_index: Option<ZIndex>,
    pub rect: Rect,
    pub manually_hidden: bool,
    pub scissor_rect_id: Option<ScissorRectID>,
}
```

1. The `<A: Clone 'static>` is a generic for the user's `Action` type. If your element emits an action then it needs this generic.
2. The action that should be emitted when the user selects this element. This can also be `None` meaning the user does not wish to emit an action.
3. Whether or not this element should be disabled when created. This "disabled" bool is our element's only custom state. (But you can of course add whatever custom state you need.)
4. A bunch of other fields that most elements should contain (with a few exceptions*). If this was C++ land we could have derived a base class to avoid duplicating this code, but alas this is crab country. 🦀

> \* If your element doesn't have a style struct, then you don't need the `class` field. Also, some special case elements such as Yarrow's `DropDownMenu`, `Tooltip`, and `FloatingTextInput` elements don't have a `rect` field in their builder since layout is always done after the fact. (Some of them also contain their own custom layout state.)

Now set up the builder pattern:

```rust
impl<A: Clone + 'static> MyCustomElementBuilder<A> {
    pub fn new() -> Self {
        Self {
            on_selected_action: None,
            disabled: false,
            class: None,
            z_index: None,
            rect: Default::default(),
            manually_hidden: false,
            scissor_rect_id: None,
        }
    }

    /// The action to emit when this element is selected (clicked or key pressed)
    pub fn on_select(mut self, action: A) -> Self {
        self.on_selected_action = Some(action);
        self
    }

    /// Whether or not this element is in the disabled state
    pub fn disabled(mut self, disabled: bool) -> Self {
        self.disabled = disabled;
        self
    }

    /// The style class ID
    ///
    /// If this method is not used, then the current class from the window context will
    /// be used.
    pub fn class(mut self, class: ClassID) -> Self {
        self.class = Some(class);
        self
    }

    /// The z index of the element
    ///
    /// If this method is not used, then the current z index from the window context will
    /// be used.
    pub fn z_index(mut self, z_index: ZIndex) -> Self {
        self.z_index = Some(z_index);
        self
    }

    /// The bounding rectangle of the element
    ///
    /// If this method is not used, then the element will have a size and position of
    /// zero and will not be visible until its bounding rectangle is set.
    pub fn rect(mut self, rect: Rect) -> Self {
        self.rect = rect;
        self
    }

    /// Whether or not this element is manually hidden
    ///
    /// By default this is set to `false`.
    pub fn hidden(mut self, hidden: bool) -> Self {
        self.manually_hidden = hidden;
        self
    }

    /// The ID of the scissoring rectangle this element belongs to.
    ///
    /// If this method is not used, then the current scissoring rectangle ID from the
    /// window context will be used.
    pub fn scissor_rect(mut self, scissor_rect_id: ScissorRectID) -> Self {
        self.scissor_rect_id = Some(scissor_rect_id);
        self
    }

    // TODO: Return the element handle.
    pub fn build(self, cx: &mut WindowContext<'_, A>) -> () {
        todo!()
    }
}
```

## Action Closures

If actions should change based on the state of your element, then consider using action closures. For example, the `Toggle` element has its action closure defined like this:

```rust
pub struct ToggleButtonBuilder<A: Clone + 'static> {
    pub action: Option<Box<dyn FnMut(bool) -> A>>,
    // ...
}

impl<A: Clone + 'static> ToggleButtonBuilder<A> {
    // ...

    /// The action that gets emitted when the user selects/toggles this element.
    ///
    /// The data in the closure contains the new toggle state
    /// (`true` = on, `false` = off).
    pub fn on_toggled<F: FnMut(bool) -> A + 'static>(mut self, f: F) -> Self {
        self.action = Some(Box::new(f));
        self
    }

    // ...
}
```

> For more information about how Rust closures work, see [https://doc.rust-lang.org/book/ch13-01-closures.html](https://doc.rust-lang.org/book/ch13-01-closures.html)
