---
title: 'Blightspire - Boomer Shooter'
date: 2025-03-07T18:22:22+01:00
image: /images/blightspire/image.png
video: /images/blightspire/Blightspire Trailer.mp4
draft: false

badges:
    - "C++"
    - "Custom Engine"
    - "CMake"
    - "Scripting"
    - "CI/CD"

meta:
  - "👥Team Project"
  - "⌛Duration: 32 weeks"
  - "💻Platforms: Windows and Steam Deck"

Summary: "
    Year long Custom Tech project where I:

    - Integrated a **scripting language (Wren)**

    - Worked on a **code quality and testing pipeline** using GitHub workflows 

    - Managed building the engine using **CMake**
    "
---

## Introduction

**Blightspire is a custom engine game for my 3rd year project at BUAS**. It is a boomer shooter that incorporates fast paced gameplay, pixelized and grainy graphics with wave-based survival! 

**[The project is on Steam, for both PC and Steam Deck](https://store.steampowered.com/app/3365920/Blightspire/)**

The code itself is also open source: [here is a link to it!](https://github.com/BredaUniversityGames/Y2024-25-PR-BB)

Our team was composed of 9 programmers and 1 designer / producer.

## What I did 

Within the team, I was assigned the role of Engine Programmer.

- **Integrated a scripting language**
- **Designed the architecture of the Engine**
- **Setup build system using (CMake)**
- Added automated tests and linting for Pull Requests (GitHub Workflows)
- Added a Unit Testing framework (GoogleTest) 

## Scripting with Wren

We needed scripting language for making gameplay code, since it would allow for **easier iteration times by avoiding recompilations of the engine**. For that I chose a lesser know scripting language called [Wren](https://github.com/wren-lang/wren).

I chose it because:
- It has syntax similar to C, reducing the barrier of entry for my colleagues that all know C++.
- Relatively fast, on par with Lua.
- Good documentation

The main USP of scripting that I implemented was runtime reloading, since **it made the experience of developing gameplay for the rest of the team much more enjoyable and responsive.** All of our [player movement](https://github.com/BredaUniversityGames/Y2024-25-PR-BB/blob/main/game/gameplay/movement.wren) and [gun logic](https://github.com/BredaUniversityGames/Y2024-25-PR-BB/blob/main/game/weapon.wren) was implemented in Wren.

<video width="440" height="320" autoplay loop muted playsinline>
  <source src="/images/blightspire/movement.mp4" type="video/mp4">
</video>
<video width="440" height="320" autoplay loop muted playsinline>
  <source src="/images/blightspire/shooting.mp4" type="video/mp4">
</video>

I used the [wrenbind17 library](https://github.com/matusnovak/wrenbind17) for creating wren bindings for C++ since it was much easier and less error prone than the C API.

Internally, **every main script represented a scene in the game**: we used one for the game and one for the main menu, transitioning between both of them when necessary. Each script defined a **Start, Shutdown and Update** that were called when the scene/script is initialized, destroyed and every frame (respectively). **Hot-reloading comes for free with this, since it is essentially the same as destroying and constructing the same script again.**

```js
class Main {
    static Start(engine) {
        // Setup input mode
        // Setup UI callbacks
        // Create player
        // Load cathedral area
        //...
    }

    static Shutdown(engine) {
        // Clear ECS
        //...
    }

    static Update(engine, dt) {
        // Update enemies
        // Player Movement
        //...
    }
}
```

I went with this approach since using wren as a frontend for the entire engine needed to be done earlier in development and implementing something like Unity's C# behaviour component scripts required too much interop between ``wren``, ``entt`` (our ECS library) and C++, as well as requiring a better engine editor.

## CMake build system

To build the codebase I used CMake. To make it organized, **I designed a system of handling each module of the engine as a separate library**, replicating the way Unreal Engine manages its modules:

<p align="center">
<img src="/images/blightspire/modules.png" alt="Centered Image">
</p>

Each module has its own public and private folders, that define the visibility of files between dependent modules. This solution **allows the engine to have cleanly defined dependencies between modules**. I created [a set of helpers for defining modules](https://github.com/BredaUniversityGames/Y2024-25-PR-BB/blob/main/scripts/cmake/code-modules.cmake), so adding one is as easy as:

```cmake
add_library(Renderer)
module_default_init(Renderer)

target_link_libraries(Renderer
    PUBLIC Core 
    PUBLIC Application # other engine modules

    PUBLIC VulkanAPI # third party libraries
)
```

## Engine module architecture

On the topic of modules, in order to add a new "manager" class to the engine, **I created a very simple [ModuleInterface](https://github.com/BredaUniversityGames/Y2024-25-PR-BB/blob/main/engine/core/public/module_interface.hpp) to implement for the Renderer, Input, Audio, etc... classes.** 

In main, we just add all the modules we need and run the engine. **[The Engine class](https://github.com/BredaUniversityGames/Y2024-25-PR-BB/blob/main/engine/core/public/engine.hpp) is simply a service locator for all the modules and manages their lifetime, retrieval and also update order.**

```c++
int main(int argc, char* argv[])
{
    MainEngine instance;

    instance
        .AddModule<ApplicationModule>()
        .AddModule<AudioModule>()
        .AddModule<RendererModule>()
        .AddModule<GameModule>()
        // More modules here...

    return instance.Run(); // Runs event loop internally
}
```

This approach has it's problems, but it is very flexible and easy to build upon. You can also easily disable certain modules in code for debugging.

## GitHub Automated Testing

To avoid main breaking every week, I also setup some [automated GitHub workflows](https://github.com/BredaUniversityGames/Y2024-25-PR-BB/blob/main/.github/workflows/pr-checks.yml) to test building the engine on both Windows and Linux and run our unit tests, as well as [another workflow](https://github.com/BredaUniversityGames/Y2024-25-PR-BB/blob/main/.github/workflows/cpp-linter.yml) for performing a linting review of a pull request:

<div class="row">
  <div style="width: 50%">
    <img src="/images/blightspire/linter.png" alt="Linter" style="width:100%">
  </div>
  <div style="width: 50%">
    <img src="/images/blightspire/build_checks.png" alt="Build Checks" style="width:100%">
  </div>
</div>