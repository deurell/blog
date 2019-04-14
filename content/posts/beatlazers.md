---
title: "Recreating Beatsabre lazers with unicorns"
date: 2019-04-14T15:56:27+02:00
draft: true
---
I've been coding professionally for over 20 years and I still learn new things every single day. One of the most fun and rewarding ways I use to get better is to recreate things in games I love. And I love many games. It never ends. An endless, lovely, gamedev tutorial!

One of the games I love is Beatsabre. I wanted to recreate it as a VR learning experience on my HTC Vive. It was a really fun journey. The first thing I needed to work out was to find a good way to slice geometries. That kinda sidetracked me for a while. But that's another beautiful story including sliced unicorns, which might just be another blog post.

![slicing unicorns](/uni.gif)

So, after the mesh slicing was in place I needed lazers. Everyone who has played Beatsabre knows that you need cubes with some kind of lazer material. Turns out creating lazer cubes is a really nice tutorial for shader newbies. We'll use the ShaderModifier source from <https://deurell.github.io/posts/scenekit-setup/>. The code for this tutorial is in in the same repository but in the lazer branch.
```
git clone git@github.com:deurell/ShaderModifierLab.git
git checkout lazer
```
The only change in the ViewController setup is that we use a cube instead of a plane.
```
let geo = SCNBox(width: 6, height: 6, length: 6, chamferRadius: 1.0)
```
After that we setup the shader to just output the UV coordinates. This is pretty much the starting point for all shader dev work, especially if we have modified the coordinate system. Outputting the UVs with X as red, Y as green and 0 as blue looks like this. You'll remember this as a sign of "Oh nice, everything seems to be ok".

![uv](/uv.png)

SceneKit calls UV's Texcoords, just like Unreal. It stashed them in a material, in this case the diffuse prop. The shaderModifier code looks like this:
```
#pragma transparent
#pragma body
float2 uv = _surface.diffuseTexcoord;
_output.color.rgb = float3(uv,0);
```
