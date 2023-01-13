---
title: "Using the RealityKit AnchorEntity"
date: 2023-01-13T11:15:08+01:00
draft: false
---
We spent last post learning how ARKit anchors work and how they relate to RealityKit entities. In the end we wrote our own RealityKit AnchorEntity based on the results of the ARSessions provided ARAnchors. This is all fine but if we just need a quick anchor we can use the RealityKit provided AnchorEntity.

We start the same way by setting up an ARSession. Requesting horizontal plane detection.
```
private func setupARSession() {
    let session = self.session
    let configuration = ARWorldTrackingConfiguration()
    configuration.planeDetection = [.horizontal]
    session.run(configuration)
}
```
When this is done we just add a RealityKit AnchorEntity to the Scene anchors collection. In order to do this it need to implement the HasAnchoring protocol. We did this ourselves in the previous post. In this case AnchorEntity already supports HasAnchoring so we can just add it. In the AnchorEntity constructor we provide what kind of ARAnchor we need for this AnchorEntity. In our case we request an horizontal plane anchor with a minimum size of 0.5m x 0.5m. 

RealityKit will not render or simulate this entity or its children until it has a valid anchor. We can check if the AnchorEntity is anchored by checking its isAnchored property. We can add our child entities to the anchor directly and wait for RealityKit to provide an anchor but it's pretty nice to be able to add them at the point when RealityKit hooks it up with a found, valid anchor. This can be done using the Combine provided SceneEvents.AnchoredStateChanged event.

When a suitable anchor is found we'll get a SceneEvents.AnchoredStateChanged event and at this point we can do our scene setup.

```
private func setupScene() {
    let anchor = AnchorEntity(.plane(.horizontal, classification: .any,
                                        minimumBounds: [0.5, 0.5]))
    scene.anchors.append(anchor)
    self.anchorEntity = anchor
    
    sub = arView.scene.subscribe(to: SceneEvents.AnchoredStateChanged.self) { event in
        let mesh = MeshResource.generateBox(size: 0.1)
        let material = SimpleMaterial(color: .blue, isMetallic: false)
        let entity = ModelEntity(mesh: mesh, materials: [material])
        self.anchorEntity?.addChild(entity)
    }
}
```
This is the base setup I use for almost all my quick prototypes. Hope this helps!

![app](/anchorentity.gif)

All code for this blog post is available [here](https://github.com/deurell/SurfaceDetection/tree/easy_anchor). Have an excellent AR dev day!