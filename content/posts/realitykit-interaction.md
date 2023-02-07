---
title: "Controlling AR-entities with an iPhone"
date: 2023-02-07T12:47:16+01:00
tags: [Swift, RealityKit]
draft: false
---
I once worked on adding support for hands tracking for an existing VR game on the Oculus Quest2. Moving from controllers to just using your hands made me think about interaction design in VR/AR in a new way. The dream scenario would be to just do the same thing as with controllers, but with hands. That would be fast, cheap and fancy, right? Turns out it worked but wasn't very fun, and didn't spark joy. Things that were easy to do with controllers were hard to do with hands and the other way around. This reminded me of the work I did porting Candy Crush to Windows desktop. Moving from mobile to desktop, from touch to mouse was the same thing. The interaction design needed to change in order to make it feel correct, nice and fun.

Now, comparing the work I did on the Oculus Quest2 using hands with running the same kind of scenario in AR on an iPhone requires an interaction model designed for this exact case. It's not using controllers, nor hands so what do you have left? Well, the best I've come up with so far is using the actual device as a controller. Controlling objects in AR using a phone is kinda like the final boss of AR interactions. Very hard to make it feel natural and fun.

So how could this look?

![move](/interaction_move.png)
![move](/device_move.gif)

To get a better feeling of movement when moving the virtual object I added a tilting behavior. Tilting the device will also tilt the virtual object.

![tilt](/device_tilt.gif)

Turns out this works pretty good, but you might need a way to do more precise movement. One solution is to use the iPhone screen as a trackpad controlling the virtual objects movement in the X/Z plane.

![move](/interaction_pan.png)
![trackpad](/device_trackpad.gif)

So a quick browse through the implementation details. I based the design on the RealityKit Entity Component System. The entity adds support for interaction by adding an InterAction component. This makes any entity with this component available for device interaction. When we grab a virtual object by touching the screen while having the object under the crosshair, we add a GrabbedComponent to the entity. This makes the Interaction system kick in, start tracking the entity and controlling it based on the iPhone device tranform in the Interaction system update loop. Pretty clean and a nice intro to the RealityKit ECS system if you ask me. :)

![move](/interaction_ecs.png)

The end result looks like this. 

![demo](/cupmover3.gif)

The Starbucks cup asset was created by Slime103. License and more info available [here](https://skfb.ly/6spIO)

All code for this blog post is available [here](https://github.com/deurell/InterAction). Have an excellent AR dev day!
