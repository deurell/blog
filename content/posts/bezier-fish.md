---
title: "Adding Bezier animations to RealityKit"
date: 2023-01-17T10:15:35+01:00
tags: [Swift, RealityKit]
draft: false
---
I love animation systems. I wrote animation systems for the C64 and Amiga in the 80's and 90's, for fashion designer apps at H&M, for games like Candy Crush and god knows how many more. RealityKit has a nice and user friendly animation system that can handle different linear world space transforms, play model space animations from usdz models and combine them in sequences or groups.
![animation-resource](/animation-resouce.jpeg)
RealityKit entities can play an AnimationResource with the playAnimation method and AnimationResources are created from AnimationDefinitions. The simplest example is a linear translate animation. 
```
let animationDefinition = FromToByAnimation(to: Transform(translation: [1, 0, 0]), 
    duration: 2.0, 
    bindTarget: .transform)
let animationResource = try! AnimationResource.generate(with: animationDefinition)
fishEntity.playAnimation(animationResource)
```
This will create an AnimationDefinition that translates the enitity one meter on the X axis in two seconds.  The AnimationDefinition is bound to a target, this target can be a property or as in this case the Transform matrix of the entity. You then create an AnimationResource based on the definition and play in on the entity.

One thing is missing though. A tool that I almost always use is simple quadratic or cubic bezier animations and there is no ready-to-go packaged way of doing this in RealityKit. You can roll your own and run it in the system game loop but it's nice to be able to handle them in the same way as all other RealityKit animations. This way you get support for fire and forget playing, grouping and sequencing animations and all the other nice things that are provided by the framework. 

The easiest way of doing this is an extension method to AnimationResource ([Extension source](https://github.com/deurell/SurfaceDetection/blob/bezier/SurfaceDetection/AnimationResource%2BBezier.swift)). RealityKit includes a SampledAnimation class that takes an array of keyframes and creates an AnimationDefinition based on those provided frames. If you set the tween mode to .linear it will interpolate between the keyframes and if you pick a nice enough number of keyframes, write a simple Bezier function to generate the frames you have Bezier animation based on the RealityKit SampledAnimation class ready to go. I wrote one and you use it like this:
```
 let bezierAnimationResource = AnimationResource.quadracticBezierAnimation(start: Constants.fishStartPosition, 
    control: [0, 0.4, 0],
    end: Constants.fishEndPosition,
    step: 0.025,
    speed: 4.7,
    timingFunction: AnimationResource.quadraticEaseInOut)
```
This makes it possible to play world space Bezier transformations on an entity while at the same time playing a usdz provided model space animation. A pretty common scenario for games and apps. Hope this helps!

![bezier-fish](/bezier-fish.gif)

All code for this blog post is available [here](https://github.com/deurell/SurfaceDetection/tree/bezier). Have an excellent AR dev day!