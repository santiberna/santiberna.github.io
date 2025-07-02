---
title: 'RusterizerS - Software Rasterizer'
date: 2024-03-22T18:22:22+01:00
image: /images/rusterizer/image.png
video: /images/rusterizer/rusterizer_video.mp4
draft: false

badges:
    - "Rust"
    - "3D Math"
    - "Graphics"

meta:
  - "👥Solo learning project"
  - "💻Platforms: Windows"
  - "⌛Duration: 1 week"

Summary: "
    Small learning exercise about building a Software Rasterizer with Rust:

    - Draws triangle with support for vertex colours and textures.

    - Implements correct triangle clipping for near plan and far plane.
    "
---

## What I learned 

- **Working with Rust language features**
- **How the graphics pipeline is implemented under the hood**
- Triangle math and barycentric coordinates
- Implementing triangle clipping
- Some popular Rust libraries for math, windowing, image loading...

## The project

The Rusterizer is a part of a collection of masterclasses that happen yearly at BUAS in between class blocks. The aim of this masterclass is to learn a bit about Rust and the math that happens behind the scenes in our GPUs.

**This is my final product at the end of the week.** [Full code can be found in my GitHub](https://github.com/santiberna/RusterizerS)

It supports:
- Triangle drawing with texture UVs and vertex colours.
- Correct near and far plane clipping.

The code is not very optimized for runtime performance and I prioritized making every shader stage as decoupled from the others (instead of placing everything in a nested for loop). This was quite helpful when implementing clipping (that has the chance to generate more triangles from one) and for understanding the entire system.

```rust
// SETUP
let mut output_surface = Texture::new(RESOLUTION_WIDTH, RESOLUTION_HEIGHT);
let mut depth_attachment = DepthTexture::new(RESOLUTION_WIDTH, RESOLUTION_HEIGHT);

//Camera Setup
let mut camera = Camera::default();
camera.aspect_ratio = (RESOLUTION_WIDTH as f32) / (RESOLUTION_HEIGHT as f32);

//Shader abstractions
let mut vertex_shader = vertex::VertexShader::default();
let mut frag_shader = fragment::FragmentShader::default();

// Texture
frag_shader.mesh_texture = load_image_file(std::path::Path::new("assets/icon.jpeg")).unwrap();

// MAIN LOOP
while window.is_open() 
{
    let (view, projection) = camera.generate_view_projection();
    vertex_shader.view = view;
    vertex_shader.projection = projection;

    //clear
    output_surface.clear(colour::f32_to_hex(1.0, 0.0, 0.0, 0.0));
    depth_attachment.clear(1.0);

    // Draw all cube faces
    for model in MODEL_MATRICES 
    { 
        vertex_shader.model = model;  
        let (vertex_output, indices) = vs.dispatch(&PLANE_VERTEX_DATA, &PLANE_INDEX_DATA);
        frag_shader.dispatch(&mut output_surface, &mut depth_attachment, &vertex_output, &indices);
    }

    window.update_with_buffer(output_surface.as_slice(), RESOLUTION_WIDTH, RESOLUTION_HEIGHT).unwrap();
}
```

[Link to vertex shader code](https://github.com/santiberna/RusterizerS/blob/main/src/renderer/vertex.rs)

[Link to fragment shader code](https://github.com/santiberna/RusterizerS/blob/main/src/renderer/fragment.rs)