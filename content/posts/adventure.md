---
title: "Crafting a Text-Based Adventure Game in Swift"
date: 2023-08-20T14:19:38+02:00
draft: false
tags: [Swift, C++, Tutorial]
---
While working on a project, I found myself crafting a super simple text-based adventure engine. Once the core mechanics were in place, it struck me that this could double as a nice Swift tutorial. So, for those who've fond memories of classics like "Zork" and "Hobbit", join me in recreating that magic. We'll employ JSON to map out our game world and then use Swift to animate our adventures.
## 1. Our Game's Blueprint: JSON
The game's structure is described in a structured JSON format. To create a new game we just feed the engine new JSON. It supports rooms, items, and paths:
```json
{
    "startingRoom": 1,
    "rooms": [
        {
            "id": 1,
            "description": "You are in the entrance hall of an ancient temple. The floor is dusty, and the atmosphere is eerily silent. Cobwebs are in every corner.",
            "paths": {
                "north": {
                    "roomID": 2,
                    "isLocked": true
                }
            },
            "items": [
                {
                    "name": "Gem",
                    "description": "A shiny blue gem that seems to emit a faint glow.",
                    "useEffects": {
                        "gemEffect": {
                            "originatingRoomID": 2,
                            "target": "Chest",
                            "action": "open",
                            "message": "You place the gem into a slot on the chest. It creaks open, revealing a stash of gold coins!"
                        }
                    }
                },
                {
                    "name": "Key",
                    "description": "A rusty old key.",
                    "useEffects": {
                        "keyEffect": {
                            "originatingRoomID": 1,
                            "target": "north",
                            "action": "unlock",
                            "message": "You hear a click as the door to the north unlocks."
                        }
                    }
                }
            ],
            "characters": []
        },
        {
            "id": 2,
            "description": "You find yourself in a grand chamber, illuminated by torches. There's an ornate chest in the middle of the room.",
            "paths": {},
            "items": [
                {
                    "name": "Chest",
                    "description": "A large, ornate chest with intricate carvings. It seems to have a slot for something.",
                    "useEffects": {}
                }
            ],
            "characters": []
        }
    ]
}

```

## 2. Translating JSON to Swift Structures
To map our JSON to Swift, we use Codable structures:

```swift
struct Path: Codable {
  var roomID: Int
  var isLocked: Bool
}

struct UseEffect: Codable {
  var originatingRoomID: Int?
  var target: String
  var action: String
  var message: String
}

struct Item: Codable {
  var name: String
  var description: String
  var useEffects: [String: UseEffect]?
}

struct Character: Codable {
  var name: String
  var dialogue: String
}

struct Room: Codable {
  var id: Int
  var description: String
  var paths: [String: Path]
  var items: [Item]
  var characters: [Character]?
}

struct GameData: Codable {
  var startingRoom: Int
  var rooms: [Room]
}
```
## 3. Parsing the Game Data: JSON to Swift
Leveraging Swift's `Codable` protocol, we can effortlessly transform JSON into Swift structures:

```swift
let decoder = JSONDecoder()
let gameData = try decoder.decode(GameData.self, from: data)
```
Here, `JSONDecoder` transforms our JSON data into the `GameData` Swift structure.

## 4. Interacting with the Game World
Our game world is alive with items to use and paths to take. Here's how we manage these interactions:
### Picking Up Items
```swift
func getItem(named itemName: String) {
    if let index = gameRooms[currentRoomID]?.items.firstIndex(where: { $0.name == itemName }) {
        if let item = gameRooms[currentRoomID]?.items.remove(at: index) {
            playerInventory.append(item)
            print("You picked up the \(itemName).")
        }
    } else {
        print("There's no \(itemName) here to pick up.")
    }
}
```
### Using Items & Traversing Paths
Items can have various effects in the game, like unlocking doors:
```swift
func useItem(named itemName: String) {
    if playerInventory.contains(where: { $0.name == itemName }) {
        print("You used the \(itemName).")
        if let useEffect = gameRooms[currentRoomID]?.items.compactMap({ $0.useEffects?[itemName] }).first {
            print(useEffect)
        } else {
            print("Nothing happens.")
        }
    } else {
        print("You don't have a \(itemName) in your inventory.")
    }
}
```
This function checks if a particular item in the player's inventory can be used in the current room.

## 5. Command Parsing Logic
Understanding and executing player's commands is the simple brain of the game. Since this is a simple engine there won't be a fancy command implementation. Just the code that felt fun and right when I typed it. Here's a look at the parsing logic:
```swift
func handleCommand(_ rawCommand: String) {
    let command = resolveAlias(rawCommand)
    let commandParts = command.split(separator: " ")

    switch commandParts[0] {
    case "look":
        if let currentRoom = gameRooms[currentRoomID] {
            lookAround(in: currentRoom)
        }
    case "use":
        useItem(named: String(commandParts[1]))
    case "go":
        go(String(commandParts[1]))
    case "inventory":
        showInventory()
    case "get":
        getItem(named: String(commandParts[1]))
    case "quit":
        print("Goodbye!")
        exit(0)
    case "talk", "to":
        talkToCharacter(named: String(commandParts[2]))
    default:
        print("I don't understand that command.")
    }
}
```
## Conclusion
By melding the principles of Swift with the nostalgia of text-based games, you've created a fresh adventure. Now, armed with this foundation, you can craft even more intricate worlds and narratives. Enjoy your coding journey!

You can find all code for this adventure [here](https://github.com/deurell/swiftadventure) packaged as a SPM package ready for coding in VSCode or XCode!

Oh... And since I've spent a lifetime writing C++, here's a C++ version of the same [game](https://github.com/deurell/simple_adventure). It uses modern C++ in a way i like and the excellent [nlohmann/json parser](https://github.com/nlohmann/json) for super easy JSON functionality. Always interesting to see the differences between the two implementations.

![Adventure](/adventure.png)