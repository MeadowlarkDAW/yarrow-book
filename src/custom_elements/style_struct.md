# Custom Elements: Style Struct

Let's start with the easiest concept: the custom style struct for your element.

The style struct simply defines how your element will be rendered. It contains properties like colors, borders, text properties, padding around content, etc. Anything that makes sense to put into a "style sheet" can go here.

> You also don't *need* a style struct if you don't want one, i.e. if your element doesn't paint anything to the screen or if you only have a single supported theme in your application.

There are no hard and fast rules on what fields to put in your style struct, however there are things you can do to make it easier for your end users to use.

## Defining the Style Struct

Let us define the style of our custom element as follows:

```rust
#[derive(Debug, Clone, PartialEq)]
pub struct MyCustomElementStyle {
    // The size of a single shape in this element (one side of a square).
    pub shape_size: f32,

    // The spacing between shapes.
    pub shape_spacing: f32,

    // The style of the quad shape.
    pub quad_style: QuadStyle,
    // The style of the quad shape when the mouse is hovered over
    // the element.
    pub quad_style_hover: QuadStyle,

    // The color of the mesh shape.
    pub mesh_color: RGBA8,
    // The color of the mesh shape when the mouse is hovered over
    // the element.
    pub mesh_color_hover: RGBA8,

    // The duration of the animation.
    pub anim_duration: Duration,
}

impl Default for MyCustomElementStyle {
    fn default() -> Self {
        Self {
            shape_size: 30.0,
            shape_spacing: 15.0,
            quad_style: QuadStyle {
                bg: Background::Solid(rgb(0xaa, 0xaa, 0xaa)),
                border: BorderStyle::default(),
                flags: Default::default(),
            },
            quad_style_hover: QuadStyle {
                bg: Background::Solid(rgb(0xee, 0xee, 0xee)),
                border: BorderStyle::default(),
                flags: Default::default(),
            },
            mesh_color: rgb(0, 200, 100),
            mesh_color_hover: rgb(0, 255, 100),
            anim_duration: Duration::from_secs(1),
        }
    }
}
```

## Fallback Properties

While the above struct is fine, if this element is meant to be easy and flexible for end users then it may be a good idea to use "fallback properties". This allows users to use  the`..Default::default()` constructor pattern in Rust and not have to duplicate properties they do not use.

While we're at it, let's set the default colors to `TRANSPARENT` so that shapes users don't use don't appear.

```rust
#[derive(Debug, Clone, PartialEq)]
pub struct MyCustomElementStyle {
    // The size of a single shape in this element (one side of a square).
    pub shape_size: f32,

    // The spacing between shapes.
    pub shape_spacing: f32,

    // The style of the quad shape.
    pub quad_style: QuadStyle,
    // The style of the quad shape when the mouse is hovered over
    // the element.
    //
    // If this is `None`, then this will fall back to `quad_style`.
    pub quad_style_hover: Option<QuadStyle>, // changed

    // The color of the mesh shape.
    pub mesh_color: RGBA8,
    // The color of the mesh shape when the mouse is hovered over
    // the element.
    //
    // If this is `None`, then this will fall back to `mesh_color`.
    pub mesh_color_hover: Option<RGBA8>,

    // The duration of the animation.
    pub anim_duration: Duration,
}

impl Default for MyCustomElementStyle {
    fn default() -> Self {
        Self {
            shape_size: 30.0,
            shape_spacing: 15.0,
            quad_style: QuadStyle::TRANSPARENT, // changed
            quad_style_hover: None, // changed
            mesh_color: color::TRANSPARENT, // changed
            mesh_color_hover: None, // changed
            anim_duration: Duration::from_secs(1),
        }
    }
}
```

> If you want to go further, you can extract the properties inside of `QuadStyle` and provide fallbacks for each individual property to make it even easier for the user to omit properties they do not use. Take a look at the [implementation of Yarrow's `Button`](https://github.com/MeadowlarkDAW/Yarrow/blob/5b14326ea41e762a7b941147d0c7d32279abc63b/src/elements/button.rs#L188) for an example.

## The `ElementStyle` Trait

But if all of the colors are set to transparent, where do we define a sane default? We also need some way to register our custom style struct to Yarrow's style system.

This is where the `ElementStyle` trait comes in.

```rust
impl ElementStyle for MyCustomElementStyle {
    const ID: &'static str = "mylib_myelement"; // 1

    fn default_dark_style() -> Self {
        Self {
            quad_style: QuadStyle {
                bg: Background::Solid(rgb(0xaa, 0xaa, 0xaa)),
                border: BorderStyle::default(),
                flags: Default::default(),
            },
            quad_style_hover: Some(QuadStyle {
                bg: Background::Solid(rgb(0xee, 0xee, 0xee)),
                border: BorderStyle::default(),
                flags: Default::default(),
            }),
            mesh_color: rgb(0, 200, 100),
            mesh_color_hover: Some(rgb(0, 255, 100)),
            ..Default::default()
        }
    }

    fn default_light_style() -> Self {
        Self {
            quad_style: QuadStyle {
                bg: Background::Solid(rgb(0x22, 0x22, 0x22)),
                border: BorderStyle::default(),
                flags: Default::default(),
            },
            quad_style_hover: Some(QuadStyle {
                bg: Background::Solid(rgb(0x33, 0x33, 0x33)),
                border: BorderStyle::default(),
                flags: Default::default(),
            }),
            mesh_color: rgb(0, 100, 50),
            mesh_color_hover: Some(rgb(0, 150, 50)),
            ..Default::default()
        }
    }
}
```

1. The unique identifier for this style struct. To avoid potential conflicts with Yarrow's built in styles and other third-party crates, prefix it with the name of your app/library.

Also notice how you can now define a separate default style for dark mode or light mode. Neat!