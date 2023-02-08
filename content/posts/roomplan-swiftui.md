---
title: "RoomPlan using SwiftUI"
date: 2023-02-08T10:05:04+01:00
tags: [Swift, SwiftUI]
draft: false
---
I spent almost a year working with VR/AR for a pretty big game company. They make some great games, mostly in Unity and for the Oculus Quest range of headsets. Some of the games needed to mark out the surrounding play area and obstacles manually in order to start playing. Really nice games but that startup process was quite tedious. When I saw the RoomPlan session at WWDC I had this idea to get rid of the manual setup and instead use one of the players iPhones, map out the surrounding area with RoomPlan, sync it with the headset and then anchor it in the game. Still think that's a great idea. :) Anyhow, the Roomplan WWDC session was based on code using UIKit/Storyboards and I find that really hard to read so I rewrote it in SwiftUI.

You can find my SwiftUI version of the Roomplan WWDC session [here](https://github.com/deurell/roomscanner).

Most of the games at that company were made in Unity. A really nice engine but it doesn't like USD assets. There is a way to load usd assets design time to the editor but that was pretty much it. 

![unity](/unity.jpeg)

I had a run converting it to other formats using Model I/O but in the end the solution was to make a custom format for the scanned room. Since Roomplan categorizes and structures all identified objects (you can peek at the Unity screenshot scene hierarchy to check out the structure), it was quite easy to make a custom format and send it directly to the headset. I never got to finish that project but it was really fun and could be really useful. In any case, it was a really nice learning experience!

Oh and my lovely dog Bowie was almost scanned in the example, he jumped out in the very last minute. He wishes you an amazing day! 

![roomplan](/roomplan.gif)
