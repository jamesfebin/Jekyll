---
layout: post
title: "The Impatient Programmer's Guide to Bevy and Rust: Chapter 5 - Let There Be Pickups"
date: 2026-01-10 00:00:00 +0000
category: rust
excerpt: "Build an inventory system and let your player collect items from the world. Learn about Resources, distance calculations, and how to make your game world interactive."
image: /assets/book_assets/chapter5/ch5.gif
og_image: /assets/book_assets/chapter5/ch5.gif
---

<style>
.tile-image {
  margin: 0 !important;
  object-fit: none !important;
  cursor: default !important;
  pointer-events: none !important;
}
</style>

By the end of this chapter, you'll have an inventory system that lets your player collect items from the world. Walk near a plant or mushroom, and it disappears into your inventory. 

> **Prerequisites**: This is Chapter 5 of our Bevy tutorial series. [Join our community](https://discord.com/invite/cD9qEsSjUH) for updates on new releases. Before starting, complete [Chapter 1: Let There Be a Player](/posts/bevy-rust-game-development-chapter-1/), [Chapter 2: Let There Be a World](/posts/bevy-rust-game-development-chapter-2/), [Chapter 3: Let The Data Flow](/posts/bevy-rust-game-development-chapter-3/), and [Chapter 4: Let There Be Collisions](/posts/bevy-rust-game-development-chapter-4/), or clone the Chapter 4 code from [this repository](https://github.com/jamesfebin/ImpatientProgrammerBevyRust) to follow along.

![Inventory System Demo]({{ "/assets/book_assets/chapter5/ch5.gif" | relative_url }})

<div style="margin: 20px 0; padding: 15px; background-color: #e3f2fd; border-radius: 8px; border-left: 4px solid #1976d2;">
<strong>Before We Begin:</strong> <em style="font-size: 14px;">I'm constantly working to improve this tutorial and make your learning journey enjoyable. Your feedback matters - share your frustrations, questions, or suggestions on <a href="https://www.reddit.com/r/bevy/" target="_blank">Reddit</a>/<a target="_blank" href="https://discord.com/invite/cD9qEsSjUH">Discord</a>/<a href="https://www.linkedin.com/in/febinjohnjames" target="_blank">LinkedIn</a>. Loved it? Let me know what worked well for you! Together, we'll make game development with Rust and Bevy more accessible for everyone.</em>
</div>

## Building the Pickup System

In Chapter 4, we made solid objects block the player. Now let's do the opposite: make items the player can walk through and collect. 

### The Pickup System

Think about how pickups work in games you've played:

1. **Detection**: "Is the player close enough to this item?"
2. **Collection**: "Remove the item from the world"
3. **Storage**: "Remember what was collected"

We'll build this in three parts:
- A `Pickable` component to mark items as collectible
- A `handle_pickups` system to detect when the player is near items
- An `Inventory` resource to track what's been collected

```d2
direction: down

Player: {
  shape: circle
  label: Player walks near item
}

Detection: {
  shape: rectangle
  label: |md
    - Calculate distance
    - Check if within radius
  |
}

Collection: {
  shape: rectangle
  label: |md
    - Despawn item entity
    - Add to inventory
  |
}

Storage: {
  shape: rectangle
  label: |md
    - Track item counts
    - Display summary
  |
}

Player -> Detection: "Distance check"
Detection -> Collection: "Within radius"
Collection -> Storage: "Update inventory"
```

### Configuring Pickup Radius

Before we build the inventory, let's add configuration for pickup detection. We already have a `config.rs` file from Chapter 4 where we centralized collision settings. Let's add pickup configuration there.

Open `src/config.rs` and you'll see it's organized into modules: `player`, `map`, `debug`. Let's add a `pickup` module:

```rust
// src/config.rs - Add this section after the player module

/// Pickup/inventory configuration
pub mod pickup {
    /// Default radius for item pickup detection (in world units)
    pub const DEFAULT_RADIUS: f32 = 40.0;
}
```

**What's a world unit?**

In Bevy, positions are measured in world units. Our tiles are 32 units wide (from `map::TILE_SIZE`), so a radius of 40 units means the player can pick up items from slightly more than one tile away. Not too far (no vacuuming items from across the map), not too close (no pixel-perfect positioning required).

```comic
left_girl_sad: I set the radius to 1000 and now I'm collecting everything on screen!
right_guy_laugh: Magneto's origin story: one typo away from world domination.
```

## Building the Inventory System

Create a new folder `src/inventory/` for our inventory module.

### Defining Item Types

What kinds of items can the player collect? In our game, we have plants, mushrooms(or something that looks like a mushroom), and tree stumps scattered around the map. Each needs a unique identifier.

Create `src/inventory/inventory.rs`:

```rust
// src/inventory/inventory.rs
use bevy::prelude::*;
use std::collections::HashMap;
use std::fmt;

use crate::config::pickup::DEFAULT_RADIUS;

/// Types of items that can be collected.
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub enum ItemKind {
    Plant1,
    Plant2,
    Plant3,
    Plant4,
    TreeStump,
}
```


### Display Names

Players don't want to see "Plant1" in their inventory. They want "Herb" or "Flower". Let's add human-readable names:

```rust
// Append to src/inventory/inventory.rs

impl ItemKind {
    pub fn display_name(&self) -> &'static str {
        match self {
            ItemKind::Plant1 => "Herb",
            ItemKind::Plant2 => "Flower",
            ItemKind::Plant3 => "Mushroom",
            ItemKind::Plant4 => "Fern",
            ItemKind::TreeStump => "Wood",
        }
    }
}

impl fmt::Display for ItemKind {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        f.write_str(self.display_name())
    }
}
```

**What's `fmt::Display` for?**

`fmt::Display`  is like a predefined contract Rust makes with your type. It's like saying, "If you implement this trait, I'll let you use `{}` in print statements."

Without it, `println!("Collected: {}", item_kind)` would fail to compile. Rust wouldn't know how to convert your `ItemKind::Plant1` into text for display. By implementing `Display`, we fulfill the contract, we tell Rust "when you need to print this, call this `fmt` method I wrote."

### The Pickable Component

Now we need a component to mark entities as collectible. When we spawn a plant in the world, we'll attach this component to say "the player can pick this up."

```rust
// Append to src/inventory/inventory.rs

/// Component marking an entity as collectible.
#[derive(Component, Debug)]
pub struct Pickable {
    pub kind: ItemKind,
    pub radius: f32,
}

impl Pickable {
    pub fn new(kind: ItemKind) -> Self {
        Self {
            kind,
            radius: DEFAULT_RADIUS,
        }
    }
}
```

**Why store `radius` per item?**

Most items use the default radius (40 units), but maybe you want a treasure chest to have a smaller radius (must be right next to it) or a glowing orb to have a larger one (magnetic pull). Storing it in the component gives you flexibility.

### The Inventory Resource

The inventory persists across the entire game session, it's not tied to any single entity. That's what Resources are for: global game state that any system can access.

```rust
// Append to src/inventory/inventory.rs

/// Resource storing collected items.
#[derive(Resource, Default, Debug)]
pub struct Inventory {
    items: HashMap<ItemKind, u32>,
}
```

**Why `HashMap<ItemKind, u32>` instead of `Vec<ItemKind>`?**

A `Vec` would store every individual item: `[Plant1, Plant1, Plant2, Plant1...]`. To count how many Plant1s you have, you'd scan the whole list. A `HashMap` stores counts directly: `{Plant1: 3, Plant2: 1}`. It also makes lookups efficient.

### Adding Items

Now let's implement the logic to add items to the inventory:

```rust
// Append to src/inventory/inventory.rs

impl Inventory {
    /// Add an item to the inventory, returns new count.
    pub fn add(&mut self, kind: ItemKind) -> u32 {
        let entry = self.items.entry(kind).or_insert(0);
        *entry += 1;
        *entry
    }

    /// Get a summary string of inventory contents.
    pub fn summary(&self) -> String {
        if self.items.is_empty() {
            return "empty".to_string();
        }

        let mut parts: Vec<String> = self
            .items
            .iter()
            .map(|(kind, count)| format!("{}: {}", kind, count))
            .collect();
        parts.sort();
        parts.join(", ")
    }
}
```

**What's `.entry(kind).or_insert(0)` doing?**

This is HashMap's "get or create" pattern. If `kind` exists in the map, `.entry(kind)` gets a mutable reference to its value. If not, `.or_insert(0)` creates it with a count of 0. Then we increment and return the new count.

**Why return the count from `add`?**

So the pickup system can log "Picked up Herb (total: 3)". It's a small quality-of-life feature that makes debugging easier.

**What's the `summary` method for?**

It formats the inventory for display: "Herb: 3, Flower: 1, Wood: 2". We sort the items alphabetically so the order is consistent. Later, you could show this in a UI panel or debug overlay.

```comic
left_guy_smile: I collected 50 herbs but forgot to save before the boss fight.
right_girl_laugh: Speedrunning heartbreak, I see.
```

## Pickup Detection System

Now for the system that actually detects when the player is near an item. Create `src/inventory/systems.rs`:

```rust
// src/inventory/systems.rs
use bevy::prelude::*;

use crate::characters::input::Player;
use super::inventory::{Pickable, Inventory};

/// System that checks for and processes item pickups.
pub fn handle_pickups(
    mut commands: Commands,
    mut inventory: ResMut<Inventory>,
    player_query: Query<&Transform, With<Player>>,
    pickables: Query<(Entity, &GlobalTransform, &Pickable)>,
) {
    let Ok(player_transform) = player_query.single() else {
        return;
    };

    let player_pos = player_transform.translation.truncate();
    let mut collected = Vec::new();

    // Check distance to each pickable
    for (entity, global_transform, pickable) in pickables.iter() {
        let item_pos = global_transform.translation().truncate();
        let distance_sq = player_pos.distance_squared(item_pos);
        
        if distance_sq <= pickable.radius * pickable.radius {
            collected.push((entity, pickable.kind));
        }
    }

    // Process collected items
    for (entity, kind) in collected {
        commands.entity(entity).despawn();
        let count = inventory.add(kind);
        info!(
            "📦 Picked up {} (total: {}) — inventory: {}",
            kind, count, inventory.summary()
        );
    }
}
```

Let's break this down step by step.

### Getting the Player Position

We query for the player's `Transform` (filtered by `With<Player>`). `.single()` returns `Result` because there might be zero or multiple players. If it fails, we exit early.

**What's `.truncate()`?**

`Transform.translation` is a `Vec3` (x, y, z). We're in 2D, so we only care about x and y. `.truncate()` converts `Vec3` to `Vec2`, dropping the z component.

### Distance Calculation

**Why `distance_squared` instead of `distance`?**

The actual distance formula is `sqrt((x2-x1)² + (y2-y1)²)`. Square root is expensive. But we're just comparing distances, so we can compare the *squared* distances instead:

- `distance < radius` is the same as `distance² < radius²`
- No square root needed, just multiplication

This is a common game dev optimization. When checking hundreds of items every frame, it adds up.

**Why collect items into a `Vec` first?**

Remember in Chapter 1 when we learned about `mut`? That was just the beginning. Rust has more rules about how you can access and modify data, enforced by something called the **borrow checker**.

**Why does Rust need a borrow checker?**

Let's start with a fundamental problem: when should we free memory? When you create a variable in most languages, it allocates memory. When you're done with it, that memory should be freed:

```d2
direction: right

Create: Create variable\n(allocate) {
  shape: rectangle
}

Use: Use variable {
  shape: rectangle
}

Free: Free variable\n(cleanup) {
  shape: rectangle
}

Create -> Use -> Free
```

**Languages like Python and JavaScript: Garbage Collection**

In Python or JavaScript, you never explicitly free variables. A *garbage collector* runs periodically, checking which variables are still being used:

```d2
direction: right

YourCode: Your Code {
  shape: rectangle
  label: |md
    **Code**


    x = [1,2,3]
    
    y = x
    
    x = None
  |
}

GC: Garbage Collector {
  shape: rectangle
  label: |md
    **Garbage Collector**
    
    Pause program
    
    Scan all variables
    
    Find unused ones
    
    Free their memory
    
    Resume program
  |
}

YourCode -> GC
```

This is convenient, you don't think about memory. But it has costs:
- **Pause times**: The garbage collector must pause your program to scan memory
- **Overhead**: It tracks every variable at runtime, using extra memory
- **Unpredictability**: You don't know when pauses happen 

**Rust's Approach: Compile-Time Checks**

Rust takes a different approach. It checks at compile time that your code follows strict rules. When the compiler sees these rules followed, it knows *exactly* when each variable's memory can be freed, no runtime tracking needed:

```rust
// Pseudo code, don't use
fn example() {
    let items = vec![1, 2, 3];  // items owns the vector
    
    process(items);              // ownership transferred to process()
                                 // items can't be used anymore
}  // Compiler inserts cleanup here - no garbage collector!
```

The borrow checker enforces rules that let the compiler figure out memory lifetimes. Let's learn these rules.

//Todo are there actually three rules or did we make this up? can you find authentic references so we are surely on track here and update it.
**The Three Borrow Checker Rules**

**Rule 1: Many readers OR one writer, never both**

You can have many immutable references OR one mutable reference, but not both at the same time.

*Why?* Imagine you're reading a map while someone is erasing and redrawing it. You might read half-old, half-new data—corruption! This rule prevents that:

```d2
direction: down

Safe: Safe (many readers) {
  shape: rectangle
  
  Read1: Read {
    shape: rectangle
  }
  
  Read2: Read {
    shape: rectangle
  }
  
  Variable: Variable {
    shape: rectangle
  }
  
  Read1 -> Variable
  Read2 -> Variable
}

Unsafe: Unsafe (reader + writer) {
  shape: rectangle
  
  Read: Read {
    shape: rectangle
  }
  
  Write: Write {
    shape: rectangle
  }
  
  Corrupted: Corrupted! {
    shape: rectangle
  }
  
  Read -> Corrupted
  Write -> Corrupted
}
```

Example code:

//Todo the following code actually compiles

```rust
let mut inventory = vec!["Herb", "Flower"];

let reader1 = &inventory;  // ✓ Immutable borrow
let reader2 = &inventory;  // ✓ Another immutable borrow is fine
println!("{:?}", reader1); // Both can read

// This won't compile:
let writer = &mut inventory;  // ✗ ERROR: Can't mutably borrow while 
                               //   immutably borrowed
```

**Rule 2: References must always be valid (no dangling pointers)**

A reference can't outlive the data it points to.

*Why?* In C++, you can create a pointer to memory that gets freed, leaving a "dangling pointer." Reading it accesses random memory—crash or worse, subtle corruption:
//The following code in d2 diagram is confusing as to when x is freed and all.
```d2
direction: right

FunctionStarts: Function starts {
  shape: rectangle
  label: |md
    int x = 5;


    int* ptr=&x;


    return ptr;
  |
}

FunctionEnds: Function ends {
  shape: rectangle
  label: |md
    x freed!


    ptr still points here
  |
}

Oops: Oops! {
  shape: rectangle
  label: |md
    Use ptr


    CRASH!
  |
}

FunctionStarts -> FunctionEnds -> Oops
```

Rust prevents this at compile time:

```rust
fn dangling() -> &String {
    let s = String::from("hello");
    &s  // ✗ ERROR: `s` doesn't live long enough
}  // s is dropped here, but we're trying to return a reference to it!
```

The compiler rejects this. You must either return the owned value or ensure the reference outlives the data.

**Rule 3: At any time, either one mutable reference OR any number of immutable references**

This is really the same as Rule 1, stated differently for clarity. You can't have `&mut` and `&` active at the same time for the same data.

*Why?* It prevents data races and aliasing bugs:

```rust
let mut count = 0;

let reader = &count;       // ✓ Immutable borrow
let writer = &mut count;   // ✗ ERROR: Can't have both

*writer += 1;              // If this compiled, reader would see
println!("{}", reader);    // inconsistent data
```

**How this helps with memory management:**

When Rust sees code following these rules, it can prove at compile time that:
- No variable is used after being freed (Rule 2)
- No variable is modified while being read (Rules 1 & 3)
- Memory can be safely freed when the owner goes out of scope

The compiler inserts cleanup code automatically, knowing it's safe. No garbage collector needed!

**How this applies to our code:**

When we write `for (entity, global_transform, pickable) in pickables.iter()`, we're borrowing the entity storage *immutably* to read through it. But `commands.entity(entity).despawn()` needs to *mutably* borrow that same storage to remove entities.

If we tried to do both at once:

```rust
// This WON'T compile!
for (entity, _, _) in pickables.iter() {  // ← Immutable borrow starts here
    commands.entity(entity).despawn();     // ← ERROR: Trying to mutably borrow!
}
```

Rust would reject this with: *"cannot borrow as mutable because it is also borrowed as immutable."*

Why? Imagine if it worked: while you're iterating (reading) the list of entities, you're also removing entities from that same list. The iterator might try to read an entity that was just deleted, causing a crash. In C++, this would compile and crash at runtime. In Rust, it doesn't even compile.

**The solution: collect-then-process**

```rust
let mut collected = Vec::new();

// Step 1: Immutable borrow - just reading
for (entity, global_transform, pickable) in pickables.iter() {
    collected.push((entity, pickable.kind));  // Copying entity IDs
}  // ← Immutable borrow ends here

// Step 2: Mutable borrow - now we can modify
for (entity, kind) in collected {
    commands.entity(entity).despawn();  // ✓ Safe! No conflicting borrows
}
```

We finish reading before we start writing. The borrow checker is satisfied, and we avoid potential crashes.

### Processing Collected Items

For each collected item:
1. **Despawn the entity**: Removes it from the world (no more rendering, no more queries)
2. **Add to inventory**: Updates the count in the HashMap
3. **Log the pickup**: Prints to console for debugging

## Wiring It Together

Create `src/inventory/mod.rs` to expose the inventory system as a plugin:

```rust
// src/inventory/mod.rs
use bevy::prelude::*;

use crate::state::GameState;

mod inventory;
mod systems;

pub use inventory::{ItemKind, Pickable, Inventory};
use systems::handle_pickups;

/// Plugin for inventory and pickup functionality.
pub struct InventoryPlugin;

impl Plugin for InventoryPlugin {
    fn build(&self, app: &mut App) {
        app.init_resource::<Inventory>()
            .add_systems(
                Update,
                handle_pickups.run_if(in_state(GameState::Playing)),
            );
    }
}
```

**What does `.init_resource::<Inventory>()` do?**

It creates the `Inventory` resource by calling `Default::default()` (which creates an empty HashMap) and registers it with Bevy. Now any system can access it with `Res<Inventory>` or `ResMut<Inventory>`.

### Integrating the Plugin

Open `src/main.rs` and add the inventory module and plugin:

```rust
// src/main.rs - Add to module declarations
mod inventory;
```

Then add the plugin to your app:

```rust
// src/main.rs - Add to the plugin chain
.add_plugins(state::StatePlugin)
.add_plugins(characters::CharactersPlugin)
.add_plugins(inventory::InventoryPlugin)  // Add this line
.add_plugins(collision::CollisionPlugin)
```

## Making Items Pickable

Now we have the infrastructure, but no items to pick up! Our procedural map generation system needs to know which decorative objects should be pickable.

### Updating the Asset Spawner

Open `src/map/assets.rs`. We'll add a method to the `SpawnableAsset` struct that marks an asset as pickable:

```rust
// src/map/assets.rs - Add this method to the impl block for SpawnableAsset

/// Make this asset a pickable item.
pub fn with_pickable(mut self, kind: ItemKind) -> Self {
    self.pickable = Some(kind);
    self
}
```

This method lets us chain `.with_pickable(ItemKind::Plant1)` when defining spawnable assets.

Next, update the `SpawnableAsset` struct to store the pickable item kind:

```rust
// src/map/assets.rs - Add this field to SpawnableAsset struct

#[derive(Clone)]
pub struct SpawnableAsset {
    sprite_name: &'static str,
    grid_offset: GridDelta,
    offset: Vec3,
    tile_type: Option<TileType>,
    pickable: Option<ItemKind>,  // Add this line
}
```

Update the constructor to initialize this field:

```rust
// src/map/assets.rs - Update the new() method

impl SpawnableAsset {
    pub fn new(sprite_name: &'static str) -> Self {
        Self {
            sprite_name,
            grid_offset: GridDelta::new(0, 0, 0),
            offset: Vec3::ZERO,
            tile_type: None,
            pickable: None,  // Add this line
        }
    }
    // ... rest of the methods
}
```

### Making the Spawner Attach Components

Now update the spawner function to actually attach `Pickable` components when entities are created. Find the `create_spawner` function in `assets.rs` and add cases for pickable items:

```rust
// src/map/assets.rs - Update create_spawner function to handle pickables

fn create_spawner(
    tile_type: Option<TileType>,
    pickable: Option<ItemKind>,
) -> fn(&mut EntityCommands) {
    match (tile_type, pickable) {
        // Pickable plants with grass tile type
        (Some(TileType::Grass), Some(ItemKind::Plant1)) => |e: &mut EntityCommands| {
            e.insert((TileMarker::new(TileType::Grass), Pickable::new(ItemKind::Plant1)));
        },
        (Some(TileType::Grass), Some(ItemKind::Plant2)) => |e: &mut EntityCommands| {
            e.insert((TileMarker::new(TileType::Grass), Pickable::new(ItemKind::Plant2)));
        },
        (Some(TileType::Grass), Some(ItemKind::Plant3)) => |e: &mut EntityCommands| {
            e.insert((TileMarker::new(TileType::Grass), Pickable::new(ItemKind::Plant3)));
        },
        (Some(TileType::Grass), Some(ItemKind::Plant4)) => |e: &mut EntityCommands| {
            e.insert((TileMarker::new(TileType::Grass), Pickable::new(ItemKind::Plant4)));
        },
        
        // Pickable tree stump
        (None, Some(ItemKind::TreeStump)) => |e: &mut EntityCommands| {
            e.insert(Pickable::new(ItemKind::TreeStump));
        },
        
        // ... existing cases for non-pickable tiles
        
        _ => |_: &mut EntityCommands| {},
    }
}
```

### Marking Props as Pickable

Finally, open `src/map/rules.rs` and find the `build_props_layer` function. This is where decorative objects like plants and stumps are defined. Add `.with_pickable()` to make them collectible:

```rust
// src/map/rules.rs - Update the plants section in build_props_layer

// Plants - make them all pickable
terrain_model_builder.create_model(
    plant_prop.clone(), 
    vec![SpawnableAsset::new("plant_1")
        .with_tile_type(TileType::Grass)
        .with_pickable(ItemKind::Plant1)  // Add this line
    ]
);
terrain_model_builder.create_model(
    plant_prop.clone(), 
    vec![SpawnableAsset::new("plant_2")
        .with_tile_type(TileType::Grass)
        .with_pickable(ItemKind::Plant2)  // Add this line
    ]
);
terrain_model_builder.create_model(
    plant_prop.clone(), 
    vec![SpawnableAsset::new("plant_3")
        .with_tile_type(TileType::Grass)
        .with_pickable(ItemKind::Plant3)  // Add this line
    ]
);
terrain_model_builder.create_model(
    plant_prop.clone(), 
    vec![SpawnableAsset::new("plant_4")
        .with_tile_type(TileType::Grass)
        .with_pickable(ItemKind::Plant4)  // Add this line
    ]
);

// Tree stumps - make one variant pickable
terrain_model_builder.create_model(
    stump_prop.clone(),
    vec![SpawnableAsset::new("tree_stump_2")
        .with_tile_type(TileType::Tree)
        .with_pickable(ItemKind::TreeStump)  // Add this line
    ],
);
```

**What's happening here?**

We're using the builder pattern to configure each spawnable asset. `.with_tile_type(TileType::Grass)` says "this is grass for collision purposes," and `.with_pickable(ItemKind::Plant1)` says "the player can collect this as a Plant1 item."

Not all tree stumps are pickable—only `tree_stump_2` gets the `.with_pickable()` call. This adds variety to the world: some stumps are just decorative obstacles, others are harvestable resources.

**What if I want some plants to be decorative (not pickable)?**

Just don't add the `Pickable` component. The pickup system only queries for entities with `Pickable`, so decorative plants are ignored.

## Testing the Inventory

Run your game:

```bash
cargo run
```

Walk near a plant or mushroom. It should disappear, and you'll see a log message:

```
📦 Picked up Herb (total: 1) — inventory: Herb: 1
📦 Picked up Flower (total: 1) — inventory: Flower: 1, Herb: 1
📦 Picked up Herb (total: 2) — inventory: Flower: 1, Herb: 2
```

```comic
left_girl_smile: I collected 50 mushrooms in 2 minutes!
right_guy_surprised: Are you playing the game or speedrunning a grocery store?
```

## What's Next?

You've built a working inventory system! But it's invisible—players can't see what they've collected. In Chapter 6, we'll build a UI overlay that displays the inventory on screen, add item icons, and create a crafting system that combines items into new ones.

For now, experiment with your pickup system:
- Try changing `DEFAULT_RADIUS` in `config.rs` to make pickups easier or harder
- Add new `ItemKind` variants for different collectibles
- Modify the `summary` format to show items in a different order

<div style="margin: 20px 0; padding: 15px; background-color: #d4edda; border-radius: 8px; border-left: 4px solid #28a745;">
<strong>Stay Tuned for Chapter 6!</strong> <br> <a href="https://discord.com/invite/cD9qEsSjUH">Join our community</a> to get notified when the next chapter drops, where we'll build a visual inventory UI and crafting system.
<br><br>
<strong>Let's stay connected! Here are some ways</strong>
<ul>
<li>Follow the project on <a href="https://github.com/jamesfebin/ImpatientProgrammerBevyRust">GitHub</a></li>
<li>Join the discussion on <a href="https://www.reddit.com/r/bevy/">Reddit</a></li>
<li>Connect with me on <a href="https://www.linkedin.com/in/febinjohnjames">LinkedIn</a> and <a href="https://x.com/heyfebin">X/Twitter</a></li>
</ul>
</div>
