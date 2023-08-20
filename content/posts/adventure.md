---
title: "Crafting a Text-Based Adventure Game in Swift"
date: 2023-08-20T14:19:38+02:00
draft: false
---
While working on a project, I found myself crafting a super simple text-based adventure engine. Once the core mechanics were in place, it struck me that this could double as a nice Swift tutorial. So, for those who've fond memories of classics like "Zork" and "Hobbit", join me in recreating that magic. We'll employ JSON to map out our game world and then use Swift to animate our adventures.

## 1. Our Game's Blueprint: JSON

The game's structure will be described in a structured JSON format:

```json
{
    "startingRoom": 1,
    "rooms": [
        {
            "id": 1,
            "description": "In a mystical forest...",
            "paths": { "north": { "roomID": 2, "isLocked": false } },
            "items": []
        },
        ...
    ]
}
```

## 2. Translating JSON to Swift Structures

To map our JSON to Swift, we use Codable structures:

```swift
struct Item: Codable {
    let name: String
    let description: String
    let useEffects: [String: UseEffect]?
}

struct UseEffect: Codable {
    let action: String
    let message: String
}

struct Path: Codable {
    let roomID: Int
    var isLocked: Bool
}

struct Room: Codable {
    let id: Int
    let description: String
    var paths: [String: Path]
    var items: [Item]
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
func pickUpItem(_ itemName: String, in roomID: Int) -> Bool {
    ...
    playerInventory.append(room.items[index])
    ...
}
```

### Using Items & Traversing Paths

Items can have various effects in the game, like unlocking doors:

```swift
func useItem(_ itemName: String) {
    ...
    switch useEffect.action {
        case "unlockDoor":
            ...
            room.paths[direction]?.isLocked = false
            ...
        default:
            print("Unexpected action.")
    }
}
```

This function checks if a particular item in the player's inventory can be used in the current room.

## 5. Command Parsing Logic

Understanding and executing player's commands is crucial. Here's a look at the parsing logic:

```swift
func handleCommand(_ command: String) {
    let tokens = command.split(separator: " ").map(String.init)
    
    switch tokens[0] {
    case "look":
        guard let room = gameRooms[currentRoomID] else { return }
        lookAround(room: room)
    case "go":
        if tokens.count < 2 { print("Go where?") }
        else {
            move(inDirection: tokens[1])
        }
    case "pick":
        if tokens.count >= 3, tokens[1] == "up" {
            let itemName = tokens.dropFirst(2).joined(separator: " ")
            if pickUpItem(itemName, in: currentRoomID) {
                print("You picked up the \(itemName).")
            } else {
                print("There's no \(itemName) here.")
            }
        }
    case "use":
        if tokens.count >= 2 {
            let itemName = tokens.dropFirst().joined(separator: " ")
            useItem(itemName)
        }
    case "inventory":
        displayInventory()
    default:
        print("I don't understand.")
    }
}
```

## Conclusion

By melding the principles of Swift with the nostalgia of text-based games, you've created a fresh adventure. Now, armed with this foundation, you can craft even more intricate worlds and narratives. Enjoy your coding journey!

You can find all code for this adventure [here](https://github.com/deurell/adventure).