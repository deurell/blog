---
title: "Realitykit Surface Detection"
date: 2023-01-03T17:23:40+01:00
tags: [Swift, RealityKit]
draft: true
---
I've spent a lot of time learning and writing code based on RealityKit. I really like the framework. It's nicely written, has a lovely tiny, lightweight ECS implementation and makes writing AR applications and games pretty straightforward. I thought I'd write down some things I've learned that might help other AR devs out there in the wild.

The first thing that comes up when writing a new RealityKit app is detecting surfaces in order to anchor virtual objects in the real world. RealityKit has several ways of doing this so I'm going to start there. My first take on this will be to use the way ARKit uses anchors with ARAnchor and show how it relates to anchors like AnchorEntity in RealityKit. Basically we'll roll our own RealityKit AnchorEntity. Entity as in the RealityKit Entity Component System (ECS).

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
In this case our custom created DetectionView inherits from ARView and is the view that will register as a delegate for the ARSession callbacks.

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
When the running ARSession finds new horizontal surfaces it'll call the didAdd method with an array of found surfaces/anchors. We'll keep track of the anchors and also update them along the way. As the ARSession finds out more about the surrounding world it'll modify the anchors/surfaces, merging them, moving, rotating or in other ways adapt them as the session learns more about the surroundings. This is important to remember, they can change any time if we continue to run our ARSession. The update method will handle just that. In this method we'll recreate the visulizing surface mesh for the found anchors or in other ways adapt them to the current world understanding. Making them better align with the real world as we move along. The didRemove method handles removed surfaces, this could be a horizontal surface on a chair that has been moved to another location. In this case we'll remove the surface from our list and delete it.

In our DetectionView we'll keep a dictionary with anchor identifiers provided by the ARAnchors in the callback as key and our custom made AnchorEntites as values. The Anchor entities will be used as anchor points in the RealityKit scene, will contain a Model/Mesh displaying the surface and they can host other ECS entities as child objects. 
```
class DetectionView: ARView, ARSessionDelegate {
    /// Dictionary with id from the ARPlaneAnchor as key and a PlaneAnchorEntity as value
    var planes = [UUID: PlaneAnchorEntity]()
```
The callbacks will follow the same startup procedure, iterating through the provided ARAnchors, making sure they are a plane (ARPlaneAnchor) and not another type of anchor. If they aren't in the dictionary already, we'll add them to the dictionary and also add them to the scene anchors array. This is where we'll add our virtual object entities as children.
```
/// Anchors have been added. Add them to the dictionary and add PlaneAnchorEntities to the scene.
func session(_ session: ARSession, didAdd anchors: [ARAnchor]) {
    for anchor in anchors {
        if let arPlaneAnchor = anchor as? ARPlaneAnchor {
            let id = arPlaneAnchor.identifier
            if planes.contains(where: {$0.key == id}) { fatalError("anchor already exists")}
            let planeAnchorEntity = PlaneAnchorEntity(arPlaneAnchor: arPlaneAnchor)
            self.scene.anchors.append(planeAnchorEntity)
            planes[id] = planeAnchorEntity
        }
    }
}
```
One thing I absolutely love with RealityKit is that it favours composition over inheritance. This follows along nicely with the design of the Entity Component System so in most cases it's easier to support a protocol and add a component instead of inheriting from a super class. This is the case for HasModel. We make our PlaneAnchorEntity implement HasModel and add a ModelComponent in the constructor. The ModelComponent will display the plane mesh using an unlit transparent material. We'll also need to copy the transform matrix from the provided ARAnchor to the transform of our Entity in order to set the position of our visual surface. The transform needs an additional translation to adjust to the center of the surface, we do to this by adding the arPlaneAnchor.center vector. A thing that has changed in iOS16 that took me a while to figure out is that the provided transform matrix from the ARAnchor used to include the Y axis rotation. Starting with iOS16 this isn't the case anymore and we need to adjust the y axis rotation with the provided arPlaneAnchor.planeExtent.rotationOnYAxis value.
```
class PlaneAnchorEntity: Entity, HasModel, HasAnchoring {
    
    @available(*, unavailable)
    required init() {
        fatalError("Not available")
    }
    
    /// Initialize the AnchorEntity. Model is provided using a ModelComponent while HasAnchoring is implemented by updating transform and mesh directly from ARSession.
    /// Adjust position to center of plane and rotate on Y with provided angle from the Anchor planeExtent.
    init(arPlaneAnchor: ARPlaneAnchor) {
        super.init()
        self.components.set(createModelComponent(arPlaneAnchor))
        self.transform.matrix = arPlaneAnchor.transform
        self.position += arPlaneAnchor.center
        self.orientation = simd_quatf(angle: arPlaneAnchor.planeExtent.rotationOnYAxis, axis: [0,1,0])
    }
    
    /// Create a model compontent with a planemesh using the size of the provided ARPlaneAnchor.
    private func createModelComponent(_ arPlaneAnchor: ARPlaneAnchor) -> ModelComponent {
        let mesh = MeshResource.generatePlane(width: arPlaneAnchor.planeExtent.width, depth: arPlaneAnchor.planeExtent.height)
        let material = UnlitMaterial(color: .lightGray.withAlphaComponent(0.5))
        let modelComponent = ModelComponent(mesh: mesh, materials: [material])
        return modelComponent
    }
    
    /// Called when the ARsession has updated the anchor. Update the transform and the mesh with provided transform/size.
    /// Adjust position to center of plane and rotate on Y with provided angle from the Anchor planeExtent.
    func didUpdate(arPlaneAnchor: ARPlaneAnchor) throws {
        self.model?.mesh = MeshResource.generatePlane(width: arPlaneAnchor.planeExtent.width, depth: arPlaneAnchor.planeExtent.height)
        self.transform.matrix = arPlaneAnchor.transform
        self.position += arPlaneAnchor.center
        self.orientation = simd_quatf(angle: arPlaneAnchor.planeExtent.rotationOnYAxis, axis: [0,1,0])
    }
}
```
The didUpdate function recreates the model surface mesh based on the provided ARAnchor plane size and follows the same pattern as described above. And that's it. We've rolled our own RealityKit Archor entity that can be added to the Scenes anchors array. They can host virtual objects as children and they also display a visualizing mesh using a ModelComponent. There are easier ways of doing anchors in RealityKit but they all use this underlying strategy behind the scenes so this helped me grok the anchor behaviors in RealityKit when I started out.

All the code this blog post is based on is available [here](https://github.com/deurell/SurfaceDetection). Have an excellent AR dev day!