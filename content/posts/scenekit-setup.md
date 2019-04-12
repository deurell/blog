---
title: "Setting up ShaderModifiers in SceneKit"
date: 2019-04-12T18:45:31+02:00
tags: [SceneKit, Shaders]
---
I <3 the Metal shader debugger in XCode. It's one of those things that feels that they've been sent from the future. I've been doing shaders in OpenGL for years and years. The last five years I've been helping a nice Swedish game company to code casual games with a lot of Candy. And Crushes. More than once I've realized that the demos and shaders I did for fun in my teens is the very thing that's putting food on the table for my kids over 100 years later. Well.. Almost 100 years later..

After last years WWDC I downloaded the new XCode with the Metal debugger and. I was completely blown away! Writing shaders in Metal is like writing ordinary C++ 14 at work but without those pesky templates. Just the way I like C++! And the debugger. The debugger is better the my standard C++ debugger at work!

I did a lot of face tracking apps with AR kit. Pretty much like this:

![face tracking](/face.gif)

I wanted to have a quick shader lab environment without an iPhone and AR stuff but still work on the face tracking SceneKit shaders on the commute to work where I only have my laptop. The quickest setup was to do a simple plane geometry in SceneKit and hook up ShaderModifiers. 

ShaderModifiers can be both vertex and framgement modifiers and hooks in to the standard SceneKit shader setup using pragmas. Pretty convenient, I use it all the time. So how do we set it up? 

Create a standard MacOS app with a view and a ViewController, set up a SceneKit scene and hook up a shader modifier this way:

```
import Cocoa
import SceneKit

class ViewController: NSViewController {
    
    var scnView: SCNView!
    var scnScene: SCNScene!
    var cameraNode: SCNNode!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        scnView = self.view as? SCNView
        scnView.showsStatistics = false
        scnView.allowsCameraControl = true
        scnView.autoenablesDefaultLighting = false
        
        scnScene = SCNScene()
        scnScene.background.contents = NSColor.gray
        scnView.scene = scnScene
        
        cameraNode = SCNNode()
        cameraNode.camera = SCNCamera()
        cameraNode.position = SCNVector3(x:0, y:0, z:12)
        scnScene.rootNode.addChildNode(cameraNode)
        
        let plane = SCNPlane(width: 20, height: 10)
        
        guard let shaderURL = Bundle.main.url(forResource: "frag", withExtension: "shader"),
            let modifier = try? String(contentsOf: shaderURL)
            else { fatalError("Can't load shader from bundle.") }
        
        plane.shaderModifiers = [.fragment: modifier]
        
        let node = SCNNode(geometry: plane)
        plane.firstMaterial?.diffuse.contents = NSImage(named: "skull")
        plane.firstMaterial?.diffuse.wrapS = SCNWrapMode.repeat;
        plane.firstMaterial?.diffuse.wrapT = SCNWrapMode.repeat;
        scnScene.rootNode.addChildNode(node)
        scnView.isPlaying = true
    }
}
```

Add a texture to the bundle, in this case it's called skulls and create a frag.shader file with the shader modifier.

```
#pragma transparent
#pragma body

float mpi = 3.1415926535897932384626433832795;
// scenekit stashes time in scn_frame
float iTime = scn_frame.time;

// grab uv coords from our material
float2 uv = _surface.diffuseTexcoord;

// we want a coords from -1 to 1 instead of standard uv coords (0-1)
float2 p = -1.0+2.0*uv;

// length and angle from center for fancy polar coord dist.
float r = length(p);
float a = atan(p.y / p.x);

//classic demo fx
float2 uvmod = float2(p.x/abs(p.y), 1/abs(p.y));
 
// uv scroll it with time
uvmod += iTime;

// get the diffuse sampler and get col at distorted uv coords
float4 texcol = u_diffuseTexture.sample(u_diffuseTextureSampler, uvmod);
_output.color.rgba = texcol;
```

The sample is a classic memory from my boy room. A nice u=x/abs(y) v=1/abs(y), but in 2019 I don't need LUTs. Nice.

![demo time](/absy.gif)

The full source code can be found here <https://github.com/deurell/ShaderModifierLab>

<3