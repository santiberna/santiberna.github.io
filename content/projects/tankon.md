---
title: 'TankOn - Multiplayer Versus Topdown Shooter'
date: 2025-02-21T18:22:22+01:00
image: /images/tankon/image.png
video: /images/tankon/TankOnMovement.mp4
draft: false

badges:
    - "C++"
    - "SDL"
    - "Networking"
    - "Boost.Asio"
    - "UI Rendering"

meta:
  - "👥Solo passion project"
  - "💻Platforms: Windows"

Summary: "
    Solo passion project where I made a:

    - Simple LAN game for 2-4 players using **Boost.Asio** and **SDL**. 

    - Also implemented **UI with text rendering and a widget hierarchy**.
    "
---

## Introduction

TankOn! is a versus topdown shooter for 2-4 players and my first study into creating multiplayer games and networking! It uses SDL as a backend and Boost.Asio for networking.

[All the code is available on my GitHub!](https://github.com/santiberna/TankOn)

## What I did 

- **Networking using Boost.Asio**
- **Patterns for gameplay messages between Server/Client**
- **UI Framework and Widgets**
- SDL Rendering, Windowing, Input and Audio
- Signals and Slots for UI

## Networking using Boost.Asio

As my first study into multiplayer games, I used [Boost.Asio](https://www.boost.org/doc/libs/1_87_0/doc/html/boost_asio.html) as my main entry point to the domain.

I created a basic [TCP connection class](https://github.com/santiberna/TankOn/blob/main/src/network/tcp/connection.hpp) that **handled message reading and writing from sockets asynchronously**. Each message is comprised of a **header describing the type and size of the message** and the **body, which is optional and is variable in size**. This is already enough to implement some basic ping functionality:

<p align="center">
<img src="/images/tankon/Pinging.png" alt="Centered Image">
</p>

**With this I learned how asynchronous programming uses completion handlers to compose basic tasks into more complex ones.**

## Server-Client behaviour and gameplay

In order to go from pinging to server-client gameplay, I created abstractions for TCP [acceptors](https://github.com/santiberna/TankOn/blob/main/src/network/tcp/acceptor.hpp) and [resolvers](https://github.com/santiberna/TankOn/blob/main/src/network/tcp/resolver.hpp) that accepted callbacks to handle connections. **This gave me great flexibility when it came to implementing a lobby system and gameplay.**

<p align="center">
<img src="/images/tankon/Lobby.png" alt="Centered Image">
</p>

The gameplay was easy to implement, although there are some smart things I did about reducing the amount of message passing:

- Each client updates their tank position **and sends it to the server, that is then bounced to other clients.**
- Since bullets travel in a straight line, **I only need to send their starting position and starting time to all the clients** instead of sending messages every frame.
- **Tank to bullet collisions are done in the server side.**

For serialization and deserialization of all this data, I used [Cereal](https://github.com/USCiLab/cereal).

<p align="center">
<video width="440" height="320" autoplay loop muted playsinline>
  <source src="/images/tankon/TankOnMovement.mp4" type="video/mp4">
</video>
</p>

## UI Widgets and Input

I explored using [Boost.Signals2](https://www.boost.org/doc/libs/1_88_0/doc/html/signals2.html) to create a flexible UI framework. The entire system uses [a tree data structure implementation](https://github.com/kpeeters/tree.hh) to represent a Menu as a tree of multiple UI nodes (that can be widgets):

<p align="center">
<img src="/images/tankon/ui_anim.gif" width="40%" alt="Centered Image">
<div align="center">Scaling up on hovered buttons (with debug lines)</div>
</p>

<p align="center">
<video width="440" height="320" autoplay loop muted playsinline>
  <source src="/images/tankon/Widgets1.mp4" type="video/mp4">
</video>
<video width="440" height="320" autoplay loop muted playsinline>
  <source src="/images/tankon/Widgets2.mp4" type="video/mp4">
</video>

<div align="center">Sliders and toggles (Left), text input boxes (Right)</div>
</p>

At a macro level, menus exist in a stack data structure where the top element is drawn (other systems might have it drawn on top of the previous ones). This is a very useful design pattern that I utilized for UI, since it fits nicely into the intuitive way of going into a menu and going back to the previous one active (pushing and popping).

```c++
// Initialization

main_menu = MakeMainMenu(*this);
settings_menu = MakeSettingsMenu(*this);
menu_stack.push(&main_menu);

// Drawing and updating

UICursorInfo ui_input {};
ui_input.cursor_position = mouse_pos;
ui_input.cursor_state = cursor;
ui_input.deltatime = deltatime;
ui_input.typed_characters = input_text;

if (!menu_stack.empty())
{
    ZoneScopedN("UI Drawing");
    menu_stack.top()->Draw(renderer, ui_input);
}
```

For drawing individual widgets, they are stored as a tree owned by the respective menu: drawing the UI is as easy as iterating the tree depth-first (Note: I did not need to do Z ordering for the simple framework I was creating):

```c++

// Defined aliases in the Menu class
using NodeTree = tr::tree<std::unique_ptr<UINode>>;
using NodeIterator = NodeTree::iterator;

void Menu::Draw(Renderer& renderer, const UICursorInfo& cursor_params)
{
    glm::vec2 frame_size = glm::vec2(renderer.GetFrameAspectRatio(), 1.0f);
    UIDrawInfo initial { frame_size * 0.5f, frame_size, colour::WHITE };

    // loop through all root ui elements
    for (auto it = elements.begin(); it != elements.end(); it = elements.next_sibling(it))
        DrawElement(renderer, it, initial, cursor_params);
}

void Menu::DrawElement(Renderer& renderer, NodeIterator iterator, const UIDrawInfo& parent_info, const UICursorInfo& cursor_params)
{
    auto& elem = **iterator;
    if (!elem.active) return;

    UIDrawInfo current_draw_info = parent_info * elem.local_transform;
    elem.Draw(renderer, current_draw_info, cursor_params);

    // Loops through all child elements
    for (auto it = elements.begin(iterator); it != elements.end(iterator); ++it)
        DrawElement(renderer, it, current_draw_info, cursor_params);
}
```