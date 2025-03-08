---
title: 'Blossom - Nature Walking Sim'
date: 2024-07-27T18:22:22+01:00
image: /images/blossom/image.png
draft: false

badges:
    - "C++"
    - "Custom Engine"
    - "Procedural Level Editor"
    - "Dear ImGui"
    - "OpenGL"
    - "Perforce"
    - "Visual Studio"
    - "University"

meta:
  - "👥Team Size: 10 (🛠️6, 🎮2, 🎨3)"
  - "⌛Duration: 16 weeks"
  - "💻Platforms: Windows and PS5"

Summary: "Team project where I worked on Level loading and Serialization, Procedural placement of objects using texture masks and perlin noise, Editor GUI for editing a scene."
---

Blossom is a game where you play as a bee collecting honey around a beautiful 3D environment. This project originally started as a custom rendering engine for nature scenes. **Developed for both PC and the PS5.**

The game is available on [Itch.io (Link)](https://buas.itch.io/blossom-engine) and the [OpenGL port is available on GitHub (Link)](https://github.com/BredaUniversityGames/Blossom) 

## Contributions

Within the team, I was assigned the role of Engine Programmer.

- **Procedural placement of Props**
- **Editor GUI for modifying editing attributes**
- **Gameplay related to Points of Interest**
- Serializing and loading scenes for release
- Model loading for GLTF files

## Perlin Noise and Density Masks

In order to design nature scenes, We needed an algorithm that would procedurally place props like flowers and trees in a terrain (doing it practically would be too much work). **We used perlin noise combined with greyscale textures** for sampling the probability that an object would be present:

<p align="center">
<img src="/images/blossom/masking.png" style="width: 75%" alt="Centered Image">
</p>

<div class="row">
  <div style="width: 50%">
    <img src="/images/blossom/flowers1.png" alt="Example 1" style="width:100%">
  </div>
  <div style="width: 50%">
    <img src="/images/blossom/flower2.png" alt="Example 2" style="width:100%">
  </div>
</div>


Combining this with a lot of other randomization parameters for size and tilt, **we got a very effective and basic way of generating landscapes.**

## Editor GUI using ImGui

**I created a robust editor for the scene with the ability to add an arbitrary number of props to a scene** and tweak a lot of the generation parameters. This was also built upon by another colleague who integrated the FastNoise library for generating density maps.

<video width="440" height="320" controls>
  <source src="/images/blossom/editor_video.mp4" type="video/mp4">
</video>
<video width="440" height="320" controls>
  <source src="/images/blossom/editing_lighting.mp4" type="video/mp4">
</video>

This also involved serializing and loading a scene at runtime, **which I also implemented** using the [Cereal library (Link)](https://github.com/USCiLab/cereal).

## The Honey Collection Minigame

Towards the end of the project I worked a bit on the gameplay. More concretely, **I created the logic for the growing sequence when the player completes the honey collection minigame and the orbiting bees around the hive as you deliver more honey**

<div class="row">
  <div style="width: 50%">
    <img src="/images/blossom/poi.gif" alt="Example 1" style="width:100%">
  </div>
  <div style="width: 50%">
    <img src="/images/blossom/orbiting.gif" alt="Example 2" style="width:100%">
  </div>
</div>