---
title: 'TankOn - Multiplayer Versus Topdown Shooter'
date: 2025-02-21T18:22:22+01:00
image: /images/tankon/image.png
draft: false

badges:
    - "C++"
    - "SDL"
    - "Networking"
    - "Boost.Asio"
    - "UI Rendering"
    - "VSCode"
    - "University"

meta:
  - "👥Solo side project"
  - "⌛Duration: 8 weeks"
  - "💻Platforms: Windows"

Summary: "Self study project where I created a simple game for 2-4 players using Boost.Asio and SDL. Also implemented UI with text rendering and a widget hierarchy."
---

TankOn! is a versus topdown shooter for 2-4 players and my first study into creating multiplayer games and networking! It uses SDL as a backend and Boost.Asio for networking.

[All the code is available on my personal GitHub! (Link)](https://github.com/santiberna/TankOn)

## What I learned

- **Networking using Boost.Asio**
- **Patterns for gameplay messages between Server/Client**
- SDL Rendering, Windowing, Input and Audio
- WIP: Signals and Slots for UI
- WIP: UI Widgets

## Networking using Boost.Asio

As my first study into multiplayer games, I used [Boost.Asio](https://www.boost.org/doc/libs/1_87_0/doc/html/boost_asio.html) as my main entry point to the domain.

I created a basic [TCP connection class (Link)](https://github.com/santiberna/TankOn/blob/main/src/network/tcp/connection.hpp) that **handled message reading and writing from sockets asynchronously**. Each message is comprised of a **header describing the type and size of the message** and the **body, which is optional and is variable in size**. This is already enough to implement some basic ping functionality:

<p align="center">
<img src="/images/tankon/Pinging.png" alt="Centered Image">
</p>

**With this I learned how asynchronous programming uses completion handlers to compose basic tasks into more complex ones.**

## Server-Client behaviour and gameplay

In order to go from pinging to server-client gameplay, I created abstractions for TCP [acceptors (Link)](https://github.com/santiberna/TankOn/blob/main/src/network/tcp/acceptor.hpp) and [resolvers (Link)](https://github.com/santiberna/TankOn/blob/main/src/network/tcp/resolver.hpp) that accepted callbacks to handle connections. **This gave me great flexibility when it came to implementing a lobby system and gameplay.**

<p align="center">
<img src="/images/tankon/Lobby.png" alt="Centered Image">
</p>

The gameplay was easy to implement, although there are some smart things I did about reducing the amount of message passing:

- Each client updates their tank position **and sends it to the server, that is then bounced to other clients.**
- Since bullets travel in a straight line, **I only need to send their starting position and starting time to all the clients** instead of sending messages every frame.
- **Tank to bullet collisions are done in the server side.**

For serialization and deserialization of all this data, I used [Cereal (Link)](https://github.com/USCiLab/cereal).

<!-- TODO: Add a video of playing TankOn here! -->

## GUI Rendering (WIP)

This section is currently in progress! Check back later!