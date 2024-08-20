# Custom Elements: Construction

Let's start by defining the actual element itself, as well as the state it will share with its handle.

> Note that we are calling this `MyCustomElementElement` and not `MyCustomElement`, and that we are not even making it public. This is because in Yarrow, it's the element handles that get the short clean name since handles are what the user interacts with.

```rust
use std::{rc::Rc, cell::RefCell};

struct MyCustomElementElement<A: Clone + 'static> {
    on_selected_action: Option<A>,
    time_animated: Duration, // 1
    shared_state: Rc<RefCell<SharedState>>, // 2
}

struct SharedState {
    disabled: bool,
}
```

1. We need a bit of internal state to keep track of the time elapsed in the animation.
2. Our shared state is wrapped in an `Rc<RefCell<T>>`. For more information on this interior mutability pattern, see [https://doc.rust-lang.org/book/ch15-05-interior-mutability.html#having-multiple-owners-of-mutable-data-by-combining-rct-and-refcellt](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html#having-multiple-owners-of-mutable-data-by-combining-rct-and-refcellt).

> Yarrow is designed to be single-threaded only for simplicity. While in theory Yarrow *could* make use of multi-threaded processing, I would rather wait and see if this kind of optimization is even necessary before adding a bunch of complexity to both the library implementation and user's custom elements.

## The Element Constructor

[TODO]