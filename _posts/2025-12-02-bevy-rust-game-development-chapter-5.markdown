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

By the end of this chapter, you'll have an inventory system that lets your player collect items from the world. Walk near a plant or mushroom, and it disappears into your inventory. Simple, satisfying, and the foundation for crafting systems, quest items, and more.

> **Prerequisites**: This is Chapter 5 of our Bevy tutorial series. [Join our community](https://discord.com/invite/cD9qEsSjUH) for updates on new releases. Before starting, complete [Chapter 1: Let There Be a Player](/posts/bevy-rust-game-development-chapter-1/), [Chapter 2: Let There Be a World](/posts/bevy-rust-game-development-chapter-2/), [Chapter 3: Let The Data Flow](/posts/bevy-rust-game-development-chapter-3/), and [Chapter 4: Let There Be Collisions](/posts/bevy-rust-game-development-chapter-4/), or clone the Chapter 4 code from [this repository](https://github.com/jamesfebin/ImpatientProgrammerBevyRust) to follow along.

![Inventory System Demo]({{ "/assets/book_assets/chapter5/ch5.gif" | relative_url }})

<div style="margin: 20px 0; padding: 15px; background-color: #e3f2fd; border-radius: 8px; border-left: 4px solid #1976d2;">
<strong>Before We Begin:</strong> <em style="font-size: 14px;">I'm constantly working to improve this tutorial and make your learning journey enjoyable. Your feedback matters - share your frustrations, questions, or suggestions on <a href="https://www.reddit.com/r/bevy/" target="_blank">Reddit</a>/<a target="_blank" href="https://discord.com/invite/cD9qEsSjUH">Discord</a>/<a href="https://www.linkedin.com/in/febinjohnjames" target="_blank">LinkedIn</a>. Loved it? Let me know what worked well for you! Together, we'll make game development with Rust and Bevy more accessible for everyone.</em>
</div>

//Todo people are not dumb, they know what's a pickupable, rewrite this question and answer.
## What Makes Something Pickable?

In Chapter 4, we built a collision system that prevents the player from walking through trees and water. But what about items you *want* to interact with? Plants, mushrooms, treasure chests—things that should disappear when you touch them and go into your inventory.

The difference is simple:
- **Collision**: "You can't go through this"
- **Pickup**: "You can collect this"

```comic
left_guy_smile: I walked into a tree and collected it!
right_girl_angry: That's not how physics works. Or inventory systems.
```

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
    **Pickup System**
    - Calculate distance
    - Check if within radius
  |
}

Collection: {
  shape: rectangle
  label: |md
    **Collection**
    - Despawn item entity
    - Add to inventory
  |
}

Storage: {
  shape: rectangle
  label: |md
    **Inventory Resource**
    - Track item counts
    - Display summary
  |
}

Player -> Detection: "Distance check"
Detection -> Collection: "Within radius"
Collection -> Storage: "Update counts"
```

## Configuring Pickup Radius

Before we build the inventory, let's add configuration for pickup detection. We already have a `config.rs` file from Chapter 4 where we centralized collision settings. Let's add pickup configuration there.

**Why centralize configuration?**

Remember from Chapter 3 when we moved character data into a `.ron` file? Same principle. Instead of scattering magic numbers like `40.0` throughout your code, you put them in one place. Want to make pickups easier? Change one number, not hunt through five files.

Open `src/config.rs` and you'll see it's organized into modules: `player`, `map`, `debug`. Let's add a `pickup` module:

```rust
// src/config.rs - Add this section after the player module

/// Pickup/inventory configuration
pub mod pickup {
    /// Default radius for item pickup detection (in world units)
    pub const DEFAULT_RADIUS: f32 = 40.0;
}
```

**What's a "world unit"?**

In Bevy, positions are measured in world units. Our tiles are 32 units wide (from `map::TILE_SIZE`), so a radius of 40 units means the player can pick up items from slightly more than one tile away. Not too far (no vacuuming items from across the map), not too close (no pixel-perfect positioning required).

```comic
left_girl_sad: I set the radius to 1000 and now I'm collecting everything on screen!
right_guy_laugh: Congratulations, you invented telekinesis!
```

## Building the Inventory System

Create a new folder `src/inventory/` with three files:

```
inventory/
├── inventory.rs  ← Item types and storage
├── systems.rs    ← Pickup detection logic
├── mod.rs        ← Plugin that ties it together
```

### Defining Item Types

What kinds of items can the player collect? In our game, we have plants, mushrooms, and tree stumps scattered around the map. Each needs a unique identifier.

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

**Why an enum instead of strings?**

You could use `String` and store `"Plant1"`, `"Plant2"`, etc. But enums are better:
- **Type safety**: The compiler prevents typos. `ItemKind::Plant1` works, `ItemKind::Plantt1` doesn't compile.
- **Memory efficiency**: An enum is just a number (0, 1, 2...), not a heap-allocated string.
- **Exhaustive matching**: When you `match` on an enum, the compiler forces you to handle all cases.

**What's `Hash` doing here?**

We'll use `ItemKind` as a key in a `HashMap` to count items. `Hash` lets Rust convert the enum into a hash value for fast lookups. Without it, you couldn't use `ItemKind` as a HashMap key.

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

**What's `&'static str`?**

The `'static` lifetime means this string lives for the entire program. `"Herb"` is a string literal baked into your binary, so it never gets freed. Returning `&'static str` is cheap—just a pointer, no allocation.

**What's `fmt::Display` for?**

This trait lets you use `{}` in format strings. Now you can write `println!("Collected: {}", item_kind)` and it prints "Herb" instead of "Plant1". The `Display` implementation just calls our `display_name` method.

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

**What's `Self` in the constructor?**

`Self` is shorthand for the type you're implementing. Inside `impl Pickable`, `Self` means `Pickable`. It's cleaner than writing `Pickable { kind, radius: DEFAULT_RADIUS }`.

### The Inventory Resource

Components attach to entities. But the inventory isn't tied to any single entity—it's global game state. That's what Resources are for.

```rust
// Append to src/inventory/inventory.rs

/// Resource storing collected items.
#[derive(Resource, Default, Debug)]
pub struct Inventory {
    items: HashMap<ItemKind, u32>,
}
```

**Why `HashMap<ItemKind, u32>` instead of `Vec<ItemKind>`?**

A `Vec` would store every individual item: `[Plant1, Plant1, Plant2, Plant1...]`. To count how many Plant1s you have, you'd scan the whole list. A `HashMap` stores counts directly: `{Plant1: 3, Plant2: 1}`. Adding an item is O(1), not O(n).

**What's `Default` doing?**

`#[derive(Default)]` generates a `Default::default()` implementation that creates an empty HashMap. When we register the resource with `.init_resource::<Inventory>()`, Bevy calls `Default::default()` to create the initial inventory.

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
left_guy_surprised: My inventory says "Herb: 47, Flower: 2"!
right_girl_laugh: Someone's been stress-collecting plants instead of fighting enemies!
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

```rust
let Ok(player_transform) = player_query.single() else {
    return;
};

let player_pos = player_transform.translation.truncate();
```

We query for the player's `Transform` (filtered by `With<Player>`). `.single()` returns `Result` because there might be zero or multiple players. If it fails, we exit early.

**What's `.truncate()`?**

`Transform.translation` is a `Vec3` (x, y, z). We're in 2D, so we only care about x and y. `.truncate()` converts `Vec3` to `Vec2`, dropping the z component.

### Distance Calculation

```rust
let distance_sq = player_pos.distance_squared(item_pos);

if distance_sq <= pickable.radius * pickable.radius {
    collected.push((entity, pickable.kind));
}
```

**Why `distance_squared` instead of `distance`?**

The actual distance formula is `sqrt((x2-x1)² + (y2-y1)²)`. Square root is expensive. But we're just comparing distances, so we can compare the *squared* distances instead:

- `distance < radius` is the same as `distance² < radius²`
- No square root needed, just multiplication

This is a common game dev optimization. When checking hundreds of items every frame, it adds up.

**Why collect items into a `Vec` first?**

We can't despawn entities while iterating over the query—Bevy's borrow checker prevents it. So we collect `(entity, kind)` pairs first, then despawn them in a second loop.

### Processing Collected Items

```rust
for (entity, kind) in collected {
    commands.entity(entity).despawn();
    let count = inventory.add(kind);
    info!(
        "📦 Picked up {} (total: {}) — inventory: {}",
        kind, count, inventory.summary()
    );
}
```

For each collected item:
1. **Despawn the entity**: Removes it from the world (no more rendering, no more queries)
2. **Add to inventory**: Updates the count in the HashMap
3. **Log the pickup**: Prints to console for debugging

**What's `info!`?**

Bevy's logging macro. It prints to the console in development builds. In release builds, you can configure it to write to a file or disable it entirely. The emoji (📦) makes it easy to spot pickup events in the log.

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

**Why `.run_if(in_state(GameState::Playing))`?**

We don't want pickups to work during the loading screen or pause menu. This filter ensures `handle_pickups` only runs when the game is in the `Playing` state. We learned about state-based scheduling in Chapter 4.

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

**Does plugin order matter?**

Sometimes. `InventoryPlugin` depends on `GameState` (from `StatePlugin`) and `Player` (from `CharactersPlugin`), so it should come after them. Bevy will warn you if there are dependency issues.

## Making Items Pickable

Now we have the infrastructure, but no items to pick up! In Chapter 2, we spawned decorative objects like plants and mushrooms. Let's make them collectible.

Find where you spawn decorative objects (likely in `src/map/decorations.rs` or similar). When spawning a plant, add the `Pickable` component:

```rust
// Example: Making a plant pickable
use crate::inventory::{Pickable, ItemKind};

commands.spawn((
    Sprite { /* sprite config */ },
    Transform::from_xyz(x, y, z),
    Pickable::new(ItemKind::Plant1),  // Add this line
));
```

Do this for each decorative object type:
- `Plant1`, `Plant2`, `Plant3`, `Plant4` → Use corresponding `ItemKind`
- `TreeStump` → Use `ItemKind::TreeStump`

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
