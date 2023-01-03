---
title: "Realitykit Surface Detection"
date: 2023-01-03T17:23:40+01:00
tags: [Swift, RealityKit]
draft: true
---
I've spent a lot of time learning and writing code based on RealityKit. I really like the framework. It's nicely written, has a lovely tiny, lightweight ECS implementation and makes writing AR applications and games pretty straightforward. That is if you're able to find some nice documentation or reference projects. That's not a super easy task, so I thought I'd write down some things I've learned that might help other AR devs out there in the wild.

The first thing that comes up when writing a new RealityKit app is detecting surfaces in order to anchor virtual objects in the real world. RealityKit has several ways of doing this so I'm going to start there. My first take on this will be to use the way ARKit uses anchors with ARAnchor and show how it relates to anchors like AnchorEntity in RealityKit. We'll roll our own RealityKit AnchorEntity, a really nice learning experience.

 Starting from ARKit and moving into RealityKit makes easier to understand how RealityKit has evolved over time. In both ARKit and RealityKit we need an ARSession to start with. This ARSession coordinates all the processes that are needed to create an AR experience. Camera control, image analysis and tracking to name a few. The ARSession feeds data to the different renderers like the RealityKit ARView, SceneKit's ARSCNView or SpriteKit's ARSKView. As we're using RealityKit we'll stick to ARView. 
 
 In order to use ARViews from SwiftUI we need to wrap it in a UIViewRepresentable.
```
struct ARContainer: UIViewRepresentable {
    func makeUIView(context: Context) -> DetectionView {
        let arView = DetectionView(frame: .zero)
        arView.setup()
        return arView
    }
    func updateUIView(_ uiView: DetectionView, context: Context) {}
}
```
In this case the DetectionView inherits from ARView and is the view that will register as a delegate for the ARSession callbacks.

 The ARSession also lets us know when it detects surfaces. We'll use those surfaces to create our anchors. The anchors we find will root our virtual objects in the real world.
 ```
/// Start ARSession and setup self as delegate.
private func setupARSession() {
        let session = self.session
        let configuration = ARWorldTrackingConfiguration()
        configuration.planeDetection = [.horizontal]
        session.run(configuration)
        session.delegate = self
    }
 ```
Every ARSession needs a configuration. In our example we'll specify that we're interested in horizontal surfaces. When the ARSession finds new surfaces we'll get a callback by registering as a delegate and use those new surfaces as anchors for virtual objects in our scene. We'll implement three methods from the ARSessionDelegate protocol to handle the callbacks.
```
func session(_ session: ARSession, didAdd anchors: [ARAnchor])
func session(_ session: ARSession, didUpdate anchors: [ARAnchor])
func session(_ session: ARSession, didRemove anchors: [ARAnchor])
```
When the running ARSession finds new horizontal surfaces it'll call the didAdd method with an array of found surfaces/anchors. We'll keep track of the anchors and also update them along the way. As the ARSession finds out more about the surrounding world it'll modify the anchors/surfaces, merging them, moving, rotating or in other ways adapt them as the session learns more about the surrounding world. This is important to remember, they can change any time if we continue to run our ARSession. The update method will handle that. In this method we'll recreate the visulizing surface mesh for the found anchors or in other ways adapt them to the current world understanding. Making them better as we move along. The didRemove method handles removed surfaces, this could be a horizontal surface on a chair that has been moved to another location. In this case we'll remove the surface from our list and delete it.

dictionary

anchorentity

All the code this blog post is based on is available [here](https://github.com/deurell/SurfaceDetection). Have an excellent AR dev day!