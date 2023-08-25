---
title: "From Prototype to the Next Phase"
date: 2023-08-25T14:05:24+02:00
draft: false
tags: [Swift, TypeHike]
---
Starting a new project often comes with the desire for perfection right from the get-go. However, I've always found immense value in beginning with a rough prototype, whether it was for my recent multiplayer Unreal 5 networking implementation gig for a First Person Shooter, fancy Candy Crush features, or that quirky AR game I spent way too much time and energy on. My latest endeavor? A super simple text adventure game engine. Let's dive into this weird journey.

### **1. The Birth of the Prototype**

Like many of my projects, this text adventure game began as a simple prototype. The aim was clear: get something functional, and quickly. The code was linear, functions were lengthy, and everything was bundled in a single file. It was the Wild West of coding, where quick fixes and hacks were the norm.

**Why start with a prototype?**
- **Rapid Development**: Witness immediate results.
- **Flexibility**: Experiment without the constraints of a rigid structure.
- **Understanding**: Grasp the core mechanics and requirements of the project.

### **2. The Need for Refactoring**

As the game's complexity grew, so did the intricacies of the prototype. Maintenance became a challenge, and introducing new features felt like navigating a lovely maze. A revamp was clearly in order.

### **3. The Evolution: Key Structural Changes**

**a. Modular Design**

- **Before**: The prototype was characterized by a monolithic structure where functions and data were all housed within a single file. This made the code harder to navigate and maintain. For instance, the game's state, command resolution, and game logic were all mixed together:

  ```swift
  var gameRooms: [Int: Room] = [:]
  var currentRoomID = 0
  var playerInventory: [Item] = []

  func resolveAlias(_ command: String) -> String {...}
  func loadGameData(from filename: String) {...}
  func lookAround(in room: Room) {...}
  ```

  This approach, while quick to implement, doesn't scale well. As the game's complexity grew, so did the challenges in managing and extending the code.

- **After**: The refactored version adopted a modular design, breaking down the game's functionalities into distinct classes. This not only made the code more organized but also more maintainable and scalable:

  - `AdventureEngine`: This class acts as the main engine driving the game. It initializes the game state, processes commands, and manages the game loop.
    ```swift
    public class AdventureEngine {
      private var gameState: GameState
      private var commandParser: CommandParser
      ...
    }
    ```

  - `CommandParser`: Responsible for parsing user input and translating it into recognizable commands for the game engine.
    ```swift
    class CommandParser {
      private var gameState: GameState
      private var commands: [String: Command] = ...
      ...
    }
    ```

  - `GameState`: This class encapsulates the current state of the game, including the current room, available rooms, and player inventory. It provides a centralized place to manage and query the game's state.
    ```swift
    class GameState {
      var currentRoomID: Int
      var gameRooms: [Int: Room]
      var playerInventory: [Item]
      ...
    }
    ```

  By segregating functionalities into specific classes, the code became more modular. Each class has a clear responsibility, making it easier to understand, debug, and extend.

**b. Command Pattern**
- **Before**: A lengthy series of `if-else` statements to decipher user commands.
  ```swift
  if command == "go" {...}
  else if command == "get" {...}
  ```
- **After**: The Command pattern was employed, segregating each command into a distinct class, streamlining the addition of new commands.
  ```swift
  struct GoCommand: Command {...}
  struct GetCommand: Command {...}
  ```

**c. Enhanced Data Structures**
- **Before**: Basic structures with limited adaptability.
  ```swift
  struct Room {
    var description: String
    var items: [String]
  }
  ```
- **After**: Introduced more comprehensive structures like `Path`, `Item`, `Character`, and others, enabling more intricate game interactions.
  ```swift
  struct Path: Codable {
    var roomID: Int
    var isLocked: Bool
  }
  struct Item: Codable {
    var name: String
    var description: String
  }
  ```

### **4. The Benefits of the New Design**

- **Scalability**: The revamped structure is primed for expansion.
- **Maintainability**: Debugging is simplified, and feature integration is streamlined.
- **Clarity**: The code's readability has been enhanced, making it a developer's delight.

### **5. Diving into the Swift Package Manager (SPM)**

The game engine is structured as an SPM package. SPM is a tool for managing the distribution of Swift code, making it easy to share and consume libraries. Here's the entire `Package.swift` file for a deeper look:

```swift
import PackageDescription

let package = Package(
    name: "TypeHike",
    products: [
        .library(
            name: "TypeHike",
            targets: ["TypeHike"]
        ),
    ],
    targets: [
        .target(
            name: "TypeHike",
            path: "Sources",
            resources: [.process("Resources")]
        )
    ]
)
```

### **6. The Takeaway**

Starting with a simple prototype is always super valuable. It served as a sandbox, allowing me to understand the game's mechanics and prerequisites. With this foundation, I could confidently transition to a more refined design. There are plenty of more refactorings to be done and functionality to add. But that's for another day. 

To my fellow developers: Dive into the initial chaos with enthusiasm. Learn from the mess, refine, and watch your code evolve towards clarity and structure. Remember, it doesn't have to be perfect from the start; it just needs to progress. Keep coding!

[![GitHub](https://img.shields.io/badge/GitHub-View%20on%20GitHub-blue)](https://github.com/deurell/TypeHike)

![Adventure](/dungeon.png)