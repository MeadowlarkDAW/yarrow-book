# Element Z Indexes

Elements do *NOT* always render in the same order they were added. Instead, elements are simply sorted by their "Z Index", with lower z index values appearing below higher z index values.

An element's z index can be set from either the `.z_index()` property in the element's builder or from the `.el.set_z_index()` method on the element's handle.

## Mouse Events

The z index also affects the order in which elements in the application receive mouse events. Elements with a higher z index value get sent a mouse event before elements with a lower z index. If an element with a higher z index "captures" the mouse event, then elements with a lower z index will not receive that mouse event.

## Builder Shorthands

Setting the z index on every single element builder can be cumbersome. Luckily, a `WindowContext` has a concept of a "z index stack" where elements that don't have a defined z index will fall back to the most recently pushed z index on that stack. For example:

```rust,ignore
pub fn my_builder(cx: &mut WindowContext<'_, MyAction>) -> Self {
    // This element will have the default z index of "0".
    let label1 = Label::builder().build(cx);

    cx.push_z_index(5);

    // These elements will have a z index of "5".
    let label2 = Label::builder().build(cx);
    let label3 = Label::builder().build(cx);

    // This element will have a z index of "20".
    let label4 = Label::builder().z_index(20).build(cx);

    // This element will have a z index of "5".
    let label5 = Label::builder().build(cx);

    cx.push_z_index(10);

    // This element will have a z index of "10".
    let label6 = Label::builder().build(cx);

    cx.pop_z_index();

    // This element will have a z index of "5".
    let label7 = Label::builder().build(cx);

    cx.pop_z_index();

    // This element will have the default z index of "0".
    let label8 = Label::builder().build(cx);
}
```

Yarrow also includes a `with_z_index` method that automatically calls `cx.push_z_index()` and `cx.pop_z_index()` for you:

```rust,ignore
pub fn my_builder(cx: &mut WindowContext<'_, MyAction>) -> Self {
    // This element will have the default z index of "0".
    let label1 = Label::builder().build(cx);

    let (label2, label3) = cx.with_z_index(5, |cx| {
        // These elements will have a z index of "5".
        let label2 = Label::builder().build(cx);
        let label3 = Label::builder().build(cx);

        (label2, label3)
    });

    // This element will have the default z index of "0".
    let label4 = Label::builder().build(cx);
}
```

There is also a `with_z_index_and_scissor_rect` method that lets you set both a z index and a scissoring rectangle ID at once.

## Performance Considerations

Currently, Yarrow treats each z index as a separate batch of primitives to send to the GPU. To keep the number of GPU draw calls and GPU memory usage down to a minimum, use as few z indexes in your application as possible.

In some cases, strategically grouping elements together into different z indexes can actually improve performance, especially with elements that have expensive text and mesh primitives. If elements that update/animate frequently are grouped into one z index and elements that rarely update are grouped into a different z index, then Yarrow/RootVG can skip preparing a batch of primitives for the rarely-updating elements whenever a frequently-updated element updates.

> An optimization may be added to Yarrow/RootVG in the future to automatically batch non-overlapping primitives together to reduce GPU draw calls and GPU memory usage (and the ability to turn this feature off for certain arbitrary z indexes). I haven't gotten around to it yet, and I want to wait and see if it's even a necessary optimization.
