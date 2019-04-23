---
title: "Metal shader development on an iPad"
date: 2019-04-23T18:53:16+02:00
tags: [Playgrounds, SceneKit, Shaders]
---
The iPad is an amazing learning device. You can read shader books on it. Draw shader ideas on it. And ever since iOS playgrounds got a major boost you can even develop Metal shaders on it using Playgrounds.

It was the only device I had with me on my easter vacation in the woods. Most of the time was spent like this:

![woods](/woods.png)

But you need to catch your breath every once in a while and what could possibly be better than coding shaders on the iPad, in the hammock?

## Enter iOS playgrounds

Playgrounds on the iPad supports SceneKit development and the shaders we wrote in the previous posts were all metal based shaders in SceneKit so we're fine! The lazer cube shader is easy, the only thing I had to do was to old school include the fragment shader modifier in a Swift string, move to UIKit and add the scene to the Playgrounds live view. The full source looks like this: 
```
import UIKit
import SceneKit
import PlaygroundSupport

let frame = CGRect(
    x: 0,
    y: 0,
    width: 400,
    height: 200)
let sceneView = SCNView(frame: frame)
sceneView.backgroundColor = #colorLiteral(red: 0.0, green: 0.0, blue: 0.0, alpha: 1.0)
sceneView.showsStatistics = false
sceneView.autoenablesDefaultLighting = false
sceneView.allowsCameraControl = true
sceneView.scene = SCNScene()

let cameraNode = SCNNode()
cameraNode.camera = SCNCamera()
cameraNode.position = SCNVector3(x: 0, y: 0, z: 12)
sceneView.scene!.rootNode.addChildNode(cameraNode)

let geo = SCNBox(
    width: 4,
    height: 4,
    length: 4,
    chamferRadius: 0.5)
let node = SCNNode(geometry: geo)
node.transform = SCNMatrix4MakeRotation(Float.pi * 0.25,1,0,0)
sceneView.scene!.rootNode.addChildNode(node)

let material = geo.firstMaterial!

let fragShader = """
#pragma transparent
#pragma arguments
float3 lazerCol;
#pragma body
float2 uv = _surface.diffuseTexcoord;
float x = 1.0-sin(uv.x*M_PI_F);
x = pow(x,4) - 0.05;
float y = 1.0-sin(uv.y*M_PI_F);
y = pow(y,4)-0.05;

float mx = mix(x,y,0.5);
float3 col = lazerCol * mx;
col *= 4.;
_output.color.rgb = col;
"""

material.shaderModifiers = [.fragment: fragShader]

material.setValue(SCNVector3(0.5, 0.8, 0.5), forKey: "lazerCol")
PlaygroundSupport.PlaygroundPage.current.liveView = sceneView
```

 
After doing that we get the reward! Metal Shader development on the iPad:

![lazer](/laz.jpeg)

It's really nice to have documentation available in the source view and Playgrounds is also pretty good at visualizing things. Like this `SCNMatrix4` rotation, rotated 45 degrees around the X-axis, inlined in the source. Fantastic for teaching!

![lazer](/laz_vi.jpeg)


## So how do we handle resources?

We really need a texture for the skull 'LUT' deformation frag shader modifier. For some reason you can't add them from the iPad. You need to boot up XCode on the Mac and add the resource to the playground from there. After that you can move back to your iPad, sync, and do an ordinary trusted

`geo.firstMaterial?.diffuse.contents = UIImage(named: "skull.jpg")`

But hey, it's pretty cool anyways. After you have the resource in place everything is back to normal.

![lazer](/sku.jpeg)

That's it. You can now fine tune your shaders on the commute to work. Or in the hammock in the deep woods of Sweden.

<3
