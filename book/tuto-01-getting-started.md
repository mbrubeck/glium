# Creating a project

To start this tutorial, we will create a new project from scratch. Even though it's highly recommended to be familiar with Rust and Cargo before starting, some little reminders are always good. Let's start by running:

```sh
cargo new --bin my_project
cd my_project
```

The directory you have just created should contain a `Cargo.toml` file which contains our project's metadata, plus a `src/main.rs` file which contains the Rust source code. If you have `src/lib.rs` file instead, that means that you forgot the `--bin` flag ; just rename the file.

In order to use the glium library, we need to add them as dependencies in our `Cargo.toml` file:

```toml
[dependencies]
glium = "*"
```

Before we can use them, we also need to import this library in our `src/main.rs` file, like this:

```rust
#[macro_use]
extern crate glium;

fn main() {
}
```

It is now time to start filling the `main` function!

# Creating a window

The first step when creating a graphical application is to create a window. If you have ever worked with OpenGL before, you know how hard it is to do this correctly. Both window creation and context creation are platform-specific, and they are sometimes weird and tedious. Fortunately, this is where the **glutin** library shines.

Initializing a window with glutin can be done by calling `glium::glutin::WindowBuilder()::new().build().unwrap()`. However we don't just want to create a *glutin* window, but a window with an OpenGL context handled by glium. Instead of calling `build()` we going to call `build_glium()`, which is defined in the `glium::DisplayBuild` trait.

```rust
fn main() {
    use glium::DisplayBuild;
    let display = glium::glutin::WindowBuilder::new().build_glium().unwrap();
}
```

But there is a problem: as soon as the window has been created, our main function exits and `display`'s destructor closes the window. To prevent this, we need to loop forever until we detect that a `Closed` event has been received:

```rust
loop {
    // listing the events produced by the window and waiting to be received
    for ev in display.poll_events() {
        match ev {
            glium::glutin::Event::Closed => return,   // the window has been closed by the user
            _ => ()
        }
    }
}
```

Right now this code will consume 100% of our CPU, but that will do for now. In a real application you should either use vertical synchronization or sleep for a few milliseconds at the end of the loop, but this is a more advanced topic.

You can now execute `cargo run`. After a few minutes during which Cargo downloads and compiles glium and its dependencies, you should see a nice little window.

### Clearing the color

The content of the window, however, is not not very appealing. Depending on your system, it can appear black, show a random image, or just some snow. We are expected to draw on the window, so the system doesn't bother initializing its color to a specific value.

Glium and the OpenGL API work similarly to a drawing software like Window's Paint or The GIMP. We start with an empty image, then draw an object on it, then another object, then another object, etc. until we are satisfied with the result. But contrary to a drawing software, you don't want your users to see the intermediate steps. Only the final result should be shown.

To handle this, OpenGL uses what is called *double buffering*. Instead of drawing directly to the window, we are drawing to an image stored in memory. Once we have finished drawing, this image is copied to the window.
This is represented in glium by the `Frame` object. When you want to start drawing something on your window, you must first call `display.draw()` in order to produce a `Frame`:

```rust
let mut target = display.draw();
```

We can then use this `target` as a drawing surface. One of the operation that OpenGL and glium provide is filling the surface with a given color. This is what we are going to do.

```rust
target.clear_color(0.0, 0.0, 1.0, 1.0);
```

Note that to use this function, we will need to import the `Surface` trait first:

```rust
use glium::Surface;
```

The four values that we pass to `clear_color` represent the four components of our color: red, green, blue and alpha. Only values between `0.0` and `1.0` are valid. Here we are drawing an opaque blue color.

Like I explained above, the user doesn't immediately see the blue color on the screen. At this point if we were in a real application, we would most likely draw our characters, their weapons, the ground, the sky, etc. But in this tutorial we will just stop here:

```rust
target.finish().unwrap();
```

This call to `finish()` means that we have finished drawing. It destroys the `Frame` object and copies our background image to the window. Our window is now filled with blue.

Here is our full `main` function after this step:

```rust
fn main() {
    use glium::{DisplayBuild, Surface};
    let display = glium::glutin::WindowBuilder::new().build_glium().unwrap();

    loop {
        let mut target = display.draw();
        target.clear_color(0.0, 0.0, 1.0, 1.0);
        target.finish().unwrap();

        for ev in display.poll_events() {
            match ev {
                glium::glutin::Event::Closed => return,
                _ => ()
            }
        }
    }
}
```
