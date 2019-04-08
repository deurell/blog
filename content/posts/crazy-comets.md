---
title: "Crazy Comets in PICO-8"
date: 2019-04-08T22:45:00+02:00
tags: [pico-8]
---

Growing up I loved to play Crazy Comets on my C64. The magic startfield seemed almost impossible to code for a 14 year old and the music by Martin Galway totally blew me away. I decided to code a pico8 remake of it together with my 6 year old daugther in order to show her how to make games. We had a fantastic time! 

![crazy comets intro](/crazy_intro.gif)

Pico8 has the simplicity needed in order to show how everything fits together. Graphics, code, music and effects. You can have a live discussion about game development while coding and showing how things are made. In my ordinary job as a C++ game developer everything takes time, sometimes a lot of time. In Lua and pico8 it's refreshingly fast. Like super, blazingly fast. And at the same time it brings the classic C64 feeling of having control. And what can I say. Kids love pixel art. A perfect teaching environment.

![crazy comets game](/crazy_game.gif)

The game is implemented as a simple state machine with objects handling the ship, comets, star field, camera and missiles. It handles collision detection with a classic AABB bounding boxes and I'll go through some of the implementation in upcoming blog posts. 

You can download the game with all source code as a cartridge here:

![cartridge](/cart.p8.png)

Or play the game in a web browser by following the following link.   
<https://lexaloffle.com/bbs/?tid=32886>

 
