---
layout: post
title: "The Impatient Programmer's Guide to Bevy and Rust: Chapter 3 - Let The Data Flow"
date: 2025-11-19 10:00:00 +0000
category: rust
excerpt: "Learn to build a data-driven character system in Bevy. We'll use a RON file to configure character attributes and animations, create a generic animation engine that handles walk, run, and jump animations, and implement character switching."
image: /assets/book_assets/chapter3/chapter3.gif
og_image: /assets/book_assets/chapter3/chapter3.gif
---

<style>
.tile-image {
  margin: 0 !important;
  object-fit: none !important;
  cursor: default !important;
  pointer-events: none !important;
}
</style>
I was supposed to release this on Halloween, but my code was so scary it kept running away. 🎃 Now that we've debugged the ghosts in the machine, let's begin!

By the end of this tutorial, you'll have a flexible, data-driven character system that supports character switching, multiple animation types (Walk, Run, Jump) configured through a data file.

> **Prerequisites**: This is Chapter 3 of our Bevy tutorial series. [Join our community](https://discord.com/invite/cD9qEsSjUH) for updates on new releases. Before starting, complete [Chapter 1: Let There Be a Player](/posts/bevy-rust-game-development-chapter-1/) and [Chapter 2: Let There Be a World](/posts/bevy-rust-game-development-chapter-2/), or clone the Chapter 2 code from [this repository](https://github.com/jamesfebin/ImpatientProgrammerBevyRust) to follow along.

![Animation System Demo]({{ "/assets/book_assets/chapter3/chapter3.gif" | relative_url }})

<div style="margin: 20px 0; padding: 15px; background-color: #e3f2fd; border-radius: 8px; border-left: 4px solid #1976d2;">
<strong>Before We Begin:</strong> <em style="font-size: 14px;">I'm constantly working to improve this tutorial and make your learning journey enjoyable. Your feedback matters - share your frustrations, questions, or suggestions on <a href="https://www.reddit.com/r/rust/comments/1p46n9c/the_impatient_programmers_guide_to_bevy_and_rust/" target="_blank">Reddit</a>/<a target="_blank" href="https://discord.com/invite/cD9qEsSjUH">Discord</a>/<a href="https://www.linkedin.com/posts/febinjohnjames_chapter-3-let-the-data-flow-continuing-activity-7397051447600664576-C3Bd/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAAlx1JIBRLKRFr1OsUTf1LYBNPYbgdfxjbc" target="_blank">LinkedIn</a>. Loved it? Let me know what worked well for you! Together, we'll make game development with Rust and Bevy more accessible for everyone.</em>
</div>

## The Problem with Hardcoded Characters

In Chapter 1, we built a player with movement and animation, however **everything was hardcoded and tightly coupled**.

```rust
// Pseudo code warning, don't use
// From Chapter 1 player.rs - Everything is hardcoded!
const TILE_SIZE ... = 64;        // ← Hardcoded sprite size
const WALK_FRAMES ... = 9;     // ← Hardcoded frame count
const MOVE_SPEED ... = 140.0;    // ← Hardcoded movement speed
const ANIM_DT ... = 0.1;         // ← Hardcoded animation timing
```

### Maintenance Nightmare

Let's see what happens when you add a second character. You'd need to duplicate everything:

```rust
// Pseudo code warning, don't use
// First character
const WARRIOR_TILE_SIZE ... = 64;
const WARRIOR_WALK_FRAMES ... = 9;
const WARRIOR_MOVE_SPEED ... = 140.0;

fn spawn_warrior(...) { /* 50 lines of code */ }
fn animate_warrior(...) { /* 80 lines of animation logic */ }
fn warrior_row_zero_based(...) { /* row mapping */ }

// Second character - DUPLICATE ALL THE CODE
const MAGE_TILE_SIZE ... = 64;
const MAGE_WALK_FRAMES ... = 6;
const MAGE_MOVE_SPEED ... = 100.0;

fn spawn_mage(...) { /* 50 lines of IDENTICAL code */ }
fn animate_mage(...) { /* 80 lines of IDENTICAL animation logic */ }
fn mage_row_zero_based(...) { /* different row mapping */ }
```

Now imagine you discover a bug in your animation system. You need to fix it in:
- `animate_warrior()`
- `animate_mage()`
- `animate_rogue()`
- `animate_paladin()`
- ...and 6 more character functions

Miss one? That character breaks. Want to add a "jump" animation? Update 10 functions. Want to change how movement works? Touch every single character's code.

This is the copy-paste maintenance nightmare you want to avoid.

```comic
left_guy_smile: I'll still copy-paste this for the other 5 characters.
right_girl_angry: Put the keyboard down and step away slowly.
```


### Data-Driven Design

The solution lies in **data-oriented programming**, a design approach where we **separate what things are (data) from what they do (behavior)**. 

Instead of tightly coupling character attributes with character-specific code, we:

**1. Separate Data from Code**

Move character properties into a single external `.ron` configuration file:

```ron
// characters.ron, All characters in one file!
(
    characters: [
        (
            name: "Warrior",
            base_move_speed: 140.0,
            max_health: 150.0,
            animations: {...}
        ),
        (
            name: "Mage",
            base_move_speed: 100.0,
            max_health: 80.0,
            animations: {...}
        ),
    ]
)
```

**What's a `.ron` file?**

RON stands for **Rusty Object Notation**, a data format similar to JSON but designed for Rust. It's human-readable, supports Rust types like tuples and structs, and allows comments. Think of it as JSON that feels native to Rust developers.

| JSON | RON |
|------|-----|
| Requires quotes on every key | Optional quotes for simple identifiers |
| No comment support | Inline and multiline comments, document your data directly |
| Trailing commas cause syntax errors | Trailing commas allowed
| Limited to JavaScript types | Native Rust types (tuples, structs, enums), matches your code |

RON eliminates JSON's verbosity while adding features that Rust developers need, making it ideal for game configuration. 

**2. Write Systems that Operate on Data**

Build generic systems that work with **any** character data:

```rust
// Pseudo code warning, don't use
// Before: Code + Data mixed together
fn animate_warrior(...) { /* hardcoded warrior logic */ }
fn animate_mage(...) { /* hardcoded mage logic */ }

// After: Flexible system that operates on data
fn animate_characters(...) { 
    // Reads character data and animates accordingly
    // Works for warrior, mage, rogue, or any future character!
}
```

**Benefits of Separation**

When data lives separately from code, the same animation system adapts to any character automatically.

Fix a bug? **One place**. Change animation speeds or frame counts? **Update the data file, no code changes**. Switch characters? **Load different data from the file**.

But the benefits extend far beyond maintenance. This separation helps with powerful capabilities that would be difficult or impossible with hardcoded values. 

You can load characters from a network server at runtime, enabling downloadable content and live game updates without redistributing your entire game binary. 

Players can create their own custom characters by simply editing the `.ron` file, opening the door to user-generated content.

**What We'll Build in This Chapter:**

While the data-driven approach opens up all these possibilities, we'll start with the foundation:
1. **Creating & loading external `.ron` file** containing all character data
2. **Generic animation system** that works with any character
3. **Runtime character switching** with number keys (1-6)
4. **Multiple animation types** (Walk, Run, Jump) per character

Ready to build this data-driven character system? Let's dive in! 

## Setting Up the Characters Module

Find the Chapter 3 project files from the [repo](https://github.com/jamesfebin/ImpatientProgrammerBevyRust/tree/main/chapter3/src/assets) and copy these spritesheets into your project’s `src/assets/` directory:


```bash
male_spritesheet.png
female_spritesheet.png
crimson_count_spritesheet.png
graveyard_reaper_spritesheet.png
lantern_warden_spritesheet.png
starlit_oracle_spritesheet.png
```

![Sprite Sheets]({{ "/assets/book_assets/chapter3/sprite_sheets.gif" | relative_url }})

### Character Schema
Every entry in `characters.ron` follows the same structure:

- `name`: Identifier that shows up in logs/UI.
- `max_health`, `base_move_speed`, `run_speed_multiplier`: Gameplay attributes.
- `texture_path`: Which spritesheet to load.
- `tile_size`: Each frame’s width/height in pixels.
- `atlas_columns`: How many columns exist in the spritesheet grid.
- `animations`: Map where the key is an `AnimationType` (`Walk`, `Run`, `Jump`) and the value is:
  - `start_row`: Row number in the spritesheet grid, counting from 0 at the top.
  - `frame_count`: Number of frames for that animation.
  - `frame_time`: Seconds per frame.
  - `directional`: `true` when the spritesheet contains four direction rows (Up, Left, Down, Right) stacked vertically for that animation. If `false`, Bevy uses the same row regardless of facing direction.

Create a folder `src/assets/characters/` and copy the `characters.ron` file from the [repo](https://github.com/jamesfebin/ImpatientProgrammerBevyRust/blob/main/chapter3/src/assets/characters/characters.ron) and add place it inside the folder. It includes data for all 6 characters.

### Setting Up the Config

With our data file in place, we need code that reads it and spawns the player. We’re going to replace the Chapter 1 `player.rs` with a new `characters` module. 

Hence delete `src/player.rs` and remove `mod player;` and `PlayerPlugin` usage from `main.rs`.

Create a new folder `src/characters/` and create the file `config.rs` inside it.
```
characters/
├── config.rs
```

First we need to define what type of animation are possible. Presently we only support `Walk`, `Run` and `Jump` animations. `AnimationDefinition` captures where each animation lives in the spritesheet, how many frames it has, and how fast it should play.

```rust
// characters/config.rs
use bevy::prelude::*;
use serde::{Deserialize, Serialize};
use std::collections::HashMap;

#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash, Serialize, Deserialize)]
pub enum AnimationType {
    Walk,
    Run,
    Jump
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct AnimationDefinition {
    pub start_row: usize,
    pub frame_count: usize,
    pub frame_time: f32,
    pub directional: bool, // true = 4 rows (one per direction), false = 1 row
}
```

**What's `Hash`, `Serialize`, and `Deserialize`?**  
`Hash` lets us use `AnimationType` as a key inside `HashMap`, so retrieving the settings for `AnimationType::Run` is just a dictionary lookup. `Serialize` and `Deserialize` allow Rust to turn these structs into `.ron` text (and back) automatically when you load or save character data.

With the animation schema in place, we can define `CharacterEntry`, the asset we load from `characters.ron`. It bundles character attributes, sprite metadata, and the animation map so every system pulls the info it needs from a single struct record.

Append the following code to `characters/config.rs`.

```rust
// Append this to characters/config.rs
#[derive(Component, Asset, TypePath, Debug, Clone, Serialize, Deserialize)]
pub struct CharacterEntry {
    pub name: String,
    pub max_health: f32,
    pub base_move_speed: f32,
    pub run_speed_multiplier: f32,
    pub texture_path: String,
    pub tile_size: u32,
    pub atlas_columns: usize,
    pub animations: HashMap<AnimationType, AnimationDefinition>,
}

impl CharacterEntry {
    pub fn calculate_max_animation_row(&self) -> usize {
        self.animations
            .values()
            .map(|def| if def.directional { def.start_row + 3 } else { def.start_row })
            .max()
            .unwrap_or(0)
    }
}

#[derive(Asset, TypePath, Debug, Clone, Serialize, Deserialize)]
pub struct CharactersList {
    pub characters: Vec<CharacterEntry>,
}
```

The `calculate_max_animation_row` helper inspects every animation definition to figure out how many rows the texture atlas needs. 

Directional animations like `Walk` often consume four stacked rows (Up, Left, Down, Right), while others, say a climb animation, may only need a single row regardless of facing. This helper keeps those differences data-driven so atlas loading code can stay generic.

**Is `self.animations` calling multiple functions sequentially?**  
Yes! This is called **method chaining**. Each function runs in order: first `.values()` gets all animation definitions from the HashMap, then `.map()` transforms each one, then `.max()` finds the largest value. They execute top-to-bottom, one after another.

**What's map doing, also is that a closure inside map?**  
`map` transforms each item in the collection. Yes, `|def|` is a **closure** as we studied in the previous chapter. It takes each animation definition (`def`) and calculates its maximum row: if the animation is directional (4 rows), it returns `start_row + 3`; otherwise just `start_row`. Think of it as "for each animation, calculate its end row."

**What's `unwrap_or`?**  
`max()` returns an `Option<usize>`, it could be `Some(number)` if there are animations, or `None` if the HashMap is empty. `unwrap_or(0)` says "if you got a number, give it to me; if you got `None`, use `0` instead." This prevents crashes when a character has no animations defined.

`CharactersList` groups all your `CharacterEntry` configs into one loadable asset, so Bevy can read every character’s data from a single JSON/RON file instead of loading many separate files.

**What's the purpose of using `Asset`, `TypePath` macros?**  
`Asset` simply tells Bevy “this struct is something you can load from disk and store in the asset server.” `TypePath` gives Bevy a unique name for the type so it knows exactly which asset you’re asking for later. Together they turn `CharacterEntry`/`CharactersList` into first-class loadable data, the same way textures or audio files already work.

**What's `HashMap<AnimationType, AnimationDefinition>` doing?**  
Each character needs different timing and sprite rows for `Walk`, `Run`, `Jump`, etc. The `HashMap` is simply a lookup table with key as `AnimationType`, so when the animation system asks for `AnimationType::Run`, it instantly receives the corresponding `AnimationDefinition` (start row, frame count, frame speed, directional flag). 


Now that we have a data structure to hold our character information, we need a system to bring it to life.


## The Animation Engine

Here we will be building the animation engine to interpret our data structure.

Create a new file `src/characters/animation.rs`. This will house the logic that brings our sprites to life.

```
characters/
├── config.rs
├── animation.rs  <- Create this
```

Our animation engine needs to help us with the following:

1. **Direction Tracking**: When your character moves right, you want them to face right. When they move up, they should face up. We need a system to convert movement into facing direction.
2. **State Management**: We need to know *when* to change animations. Did the player just start running? Just stop? Just jump? These transitions are when we reset the animation.
3. **Frame Calculation**: Given a character's current state ("running left"), which exact frame from the spritesheet should we display right now?

Let's build each piece.

### Direction Tracking

![Sprite Sheets]({{ "/assets/book_assets/chapter3/walk.gif" | relative_url }})

When your player presses the arrow keys, we get a velocity vector like `Vec2 { x: 1.0, y: 0.0 }` for moving right. But our spritesheet doesn't understand vectors—it has specific rows for Up, Down, Left, and Right animations.

We need to translate "moving in this direction" into "show this specific row of sprites." That's what the `Facing` enum does.

Add this to `src/characters/animation.rs`:

```rust
// src/characters/animation.rs
use bevy::prelude::*;
use serde::{Deserialize, Serialize};
use crate::characters::config::{CharacterEntry, AnimationType};

// Default animation timing (10 FPS = 0.1 seconds per frame)
pub const DEFAULT_ANIMATION_FRAME_TIME: f32 = 0.1;

#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash, Serialize, Deserialize)]
pub enum Facing {
    Up,
    Left,
    Down,
    Right,
}

impl Facing {
    // Convert a velocity vector into a discrete direction
    pub fn from_direction(direction: Vec2) -> Self {
        if direction.x.abs() > direction.y.abs() {
            if direction.x > 0.0 { Facing::Right } else { Facing::Left }
        } else {
            if direction.y > 0.0 { Facing::Up } else { Facing::Down }
        }
    }
    
    // Helper to map direction to row offset (0, 1, 2, 3)
    fn direction_index(self) -> usize {
        match self {
            Facing::Up => 0,
            Facing::Left => 1,
            Facing::Down => 2,
            Facing::Right => 3,
        }
    }
}
```

**Converting Movement to Facing**

The `from_direction` function takes a velocity vector and figures out which direction is dominant. If the player is moving diagonally (both x and y are non-zero), we pick the stronger component. Moving mostly right with a bit of up? Face right. Moving mostly up with a bit of right? Face up. This ensures your character always faces the most relevant direction during gameplay.

**Mapping Direction to Sprite Rows**

Our spritesheets follow a convention. For directional animations like "Walk", the rows are organized as:
- Row 0: Walk Up
- Row 1: Walk Left
- Row 2: Walk Down
- Row 3: Walk Right

The `direction_index` function converts our `Facing` enum into these row offsets (0, 1, 2, 3). So when we know the player is facing `Down` and we want to play the "Walk" animation starting at row 8, we calculate: `8 + Facing::Down.direction_index()` = `8 + 2` = row 10. That's where the "Walk Down" frames live in the atlas.

<img src="/assets/book_assets/chapter3/graveyard_reaper_spritesheet.png" alt="Walk Spritesheet" style="display: block; margin: 0 auto; width: 50%;" />

### Tracking Animation State

We need to know *when* to change animations. If your character transitions from standing still to running, we need to detect that moment and restart the animation from frame 0. Otherwise, the run animation might start mid-cycle, looking janky.

We need these components to track the *current* state of the animation. 

- **`AnimationController`**:  It knows what animation we *want* to play (e.g., "Run") and which way we are facing.
- **`AnimationState`**: It tracks if we are moving or jumping, and importantly, if we *were* moving last frame. This helps us detect state *changes* (like starting to run) so we can reset the animation timer.
- **`AnimationTimer`**: Controls how fast frames update.

Add these components to `src/characters/animation.rs`:

```rust
// Append this to src/characters/animation.rs
// Component that holds animation configuration
#[derive(Component)]
pub struct AnimationController {
    pub current_animation: AnimationType,
    pub facing: Facing,
}

impl Default for AnimationController {
    fn default() -> Self {
        Self {
            current_animation: AnimationType::Walk,
            facing: Facing::Down,
        }
    }
}

#[derive(Component, Default)]
pub struct AnimationState {
    pub is_moving: bool,
    pub was_moving: bool,
    pub is_jumping: bool,
    pub was_jumping: bool,
}

#[derive(Component, Deref, DerefMut)]
pub struct AnimationTimer(pub Timer);
```

<div style="margin: 20px 0; padding: 15px; background-color: #f8d7da; border-radius: 8px; border-left: 4px solid #dc3545;">
<em>You might notice we're using boolean flags (<code>is_moving</code>, <code>was_moving</code>, <code>is_jumping</code>, <code>was_jumping</code>) to track animation state. While this works for our current simple case, it's not the best approach for managing state transitions.As your game grows more complex (adding attacks, dodges, climbing, etc.), managing all these boolean combinations becomes error-prone and hard to maintain. 
<br><br>
In Chapter 4, we'll learn about the <strong>state machine design pattern</strong>, which provides a much cleaner and more scalable way to handle state transitions. 

For now, we'll use this simpler approach to keep our focus on understanding the animation system fundamentals.</em>
</div>

### Frame Calculation

Now we know *which* animation to play (Walk, Run, Jump) and *which* direction the character is facing. But how do we translate that into "show frame 47 from the texture atlas right now"?

Think of the spritesheet as a numbered grid, reading left-to-right, top-to-bottom. If "Walk Down" is on row 2 and the grid has 12 columns, the first frame of that animation is at position 24 (because we skip the first 2 rows: 2 × 12 = 24). If the animation has 6 frames, we'll cycle through positions 24, 25, 26, 27, 28, 29, then loop back to 24.

We'll create an `AnimationClip` struct to handle this math for us.

```rust
// Append this to src/characters/animation.rs
// Runtime animation clip helper
#[derive(Clone, Copy)]
pub struct AnimationClip {
    first: usize,
    last: usize,
}

impl AnimationClip {
    pub fn new(row: usize, frame_count: usize, atlas_columns: usize) -> Self {
        let first = row * atlas_columns;
        Self {
            first,
            last: first + frame_count - 1,
        }
    }
    
    pub fn start(self) -> usize {
        self.first
    }
    
    // Check if a frame index belongs to this clip
    pub fn contains(self, index: usize) -> bool {
        (self.first..=self.last).contains(&index)
    }
    
    // Calculate the next frame, looping back to start if needed
    pub fn next(self, index: usize) -> usize {
        if index >= self.last {
            self.first
        } else {
            index + 1
        }
    }
    
    // Check if animation has completed (used for non-looping animations like Jump)
    pub fn is_complete(self, current_index: usize, timer_finished: bool) -> bool {
        current_index >= self.last && timer_finished
    }
}
```

The `AnimationClip` struct stores just two numbers: the `first` and `last` frame indices for a specific animation sequence. The `new` method calculates these indices from the row, frame count, and atlas width. 

The `start` method returns where the animation begins. The `contains` method checks if a given frame index belongs to this clip (useful for detecting if we've wandered into the wrong animation). The `next` method advances to the next frame, automatically looping back to the start when we reach the end.

 The `is_complete` method checks if we've reached the last frame and the timer has finished—this is crucial for non-looping animations like Jump, where we need to know when to transition back to Walk.

**Connecting Clips to Controllers**

Now that we have a way to represent frame ranges, we need to connect it to our `AnimationController`. Remember, the controller knows *what* animation to play ("Run") and *which* direction we're facing ("Left"). We'll add a helper method that combines this information with the character's configuration data to produce the correct `AnimationClip`.

Now we can add a method to `AnimationController` to easily get the current clip based on the character's config:

```rust
// Append this to src/characters/animation.rs
impl AnimationController {
    pub fn get_clip(&self, config: &CharacterEntry) -> Option<AnimationClip> {
        // 1. Get the definition (e.g. "Walk" data)
        let def = config.animations.get(&self.current_animation)?;
        
        // 2. Calculate the actual row based on facing direction
        let row = if def.directional {
            def.start_row + self.facing.direction_index()
        } else {
            def.start_row
        };
        
        // 3. Create the clip
        Some(AnimationClip::new(row, def.frame_count, config.atlas_columns))
    }
}
```

### Animating Characters

Finally, the system that ties it all together. This system runs every frame and:
1. Checks if the animation changed (e.g., started moving).
2. If changed, resets the timer and frame index.
3. If not changed, ticks the timer and advances the frame.

```rust
// Append this to src/characters/animation.rs
// Generic animation system - works for ALL entities
pub fn animate_characters(
    time: Res<Time>,
    mut query: Query<(
        &AnimationController,
        &AnimationState,
        &mut AnimationTimer,
        &mut Sprite,
        &CharacterEntry,
    )>,
) {
    for (animated, state, mut timer, mut sprite, config) in query.iter_mut() {
        
        let Some(atlas) = sprite.texture_atlas.as_mut() else { continue; };
        
        // Get the correct clip for current state/facing
        let Some(clip) = animated.get_clip(config) else { continue; };
        
        // Get timing info
        let Some(anim_def) = config.animations.get(&animated.current_animation) else { continue; };
        
        // Safety: If we somehow ended up on a frame outside our clip, reset.
        if !clip.contains(atlas.index) {
            atlas.index = clip.start();
            timer.0.reset();
        }
        
        // Detect state changes
        let just_started_moving = state.is_moving && !state.was_moving;
        let just_stopped_moving = !state.is_moving && state.was_moving;
        let just_started_jumping = state.is_jumping && !state.was_jumping;
        let just_stopped_jumping = !state.is_jumping && state.was_jumping;
        
        let should_animate = state.is_jumping || state.is_moving;
        let animation_changed = just_started_moving || just_started_jumping 
                              || just_stopped_moving || just_stopped_jumping;
        
        if animation_changed {
            // Reset animation
            atlas.index = clip.start();
            timer.0.set_duration(std::time::Duration::from_secs_f32(anim_def.frame_time));
            timer.0.reset();
        } else if should_animate {
            // Advance animation
            timer.tick(time.delta());
            if timer.just_finished() {
                atlas.index = clip.next(atlas.index);
            }
        } else {
            // When idle (not moving or jumping), stay on frame 0
            if atlas.index != clip.start() {
                atlas.index = clip.start();
            }
        }
    }
}

// Helper to update "was_moving" flags at the end of the frame
pub fn update_animation_flags(mut query: Query<&mut AnimationState>) {
    for mut state in query.iter_mut() {
        state.was_moving = state.is_moving;
        state.was_jumping = state.is_jumping;
    }
}

```

**How the Animation System Works**

The animation system has three branches that handle different scenarios:

1. **Animation changed** (starting/stopping movement or jumping): Reset to frame 0 and update the timer duration for the new animation.
2. **Should animate** (actively moving or jumping): Tick the timer and advance through frames.
3. **Idle state** (standing still): Ensure we stay on frame 0, the neutral standing pose.

**Why State Change Detection Matters**

The first branch relies on detecting the *exact moment* a state changes. `is_moving` tells us the current state ("am I moving right now?"), while `was_moving` tells us the previous frame's state ("was I moving last frame?"). When `is_moving` is true but `was_moving` is false, we know the player *just* pressed a movement key.

This detection is crucial for smooth transitions. Without it, animations would continue from wherever they left off. Imagine your character's walk cycle is on frame 5, then you press jump—without resetting, the jump animation would start at frame 5 instead of frame 0, looking broken.

The third branch (idle state) handles a different case: when the player *stops* moving, we transition to Walk animation but need to ensure we display the idle pose (frame 0), not whatever frame the walk cycle was on when they stopped.

```d2
Idle: {
  shape: circle
}

Walk: {
  shape: circle
}

Run: {
  shape: circle
}

Jump: {
  shape: circle
}

Idle -> Walk: press arrow key
Walk -> Idle: release arrow key
Walk -> Run: hold shift
Run -> Walk: release shift
Idle -> Jump: press space
Walk -> Jump: press space
Run -> Jump: press space
Jump -> Idle: animation complete
```


```comic
left_guy_surprised: Why is he running while standing still?
right_girl_laugh: He's just really excited to be idle.
```

**Why do we need `update_animation_flags`?**<br>
We need `update_animation_flags` to run *after* all logic is done, so that in the *next* frame, `was_moving` correctly reflects the previous frame's state. This allows us to detect the exact moment a state changes.

## The Movement System
![Sprite Sheets]({{ "/assets/book_assets/chapter3/run.gif" | relative_url }})

Our animation engine can display the right frames, but it needs to know *what* the player is doing. The `AnimationController` we built earlier stores the *current* animation state ("I'm running left"), but something needs to *update* that state based on player input. That's where the movement system comes in. It reads keyboard input, moves the character, and tells the `AnimationController` which animation to play.

Create a new file `src/characters/movement.rs`:

```
characters/
├── config.rs
├── animation.rs
├── movement.rs  <- Create this
```

The movement system has three responsibilities:

1. **Input Reading**: Convert arrow key presses into a direction vector
2. **Movement Calculation**: Apply speed and delta time to move the character smoothly
3. **Animation Coordination**: Tell the animation system when to switch between Walk, Run, and Jump

Here's how we'll tackle this:

### Reading Player Input

When the player presses arrow keys, we need to convert those discrete button presses into a continuous direction vector. If they press Up and Right simultaneously, we want `Vec2 { x: 1.0, y: 1.0 }` for diagonal movement.

Add this to `src/characters/movement.rs`:

```rust
// src/characters/movement.rs
use bevy::prelude::*;
use crate::characters::animation::*;
use crate::characters::config::{CharacterEntry, AnimationType};

/// Read directional input and return a direction vector
fn read_movement_input(input: &ButtonInput<KeyCode>) -> Vec2 {
    const MOVEMENT_KEYS: [(KeyCode, Vec2); 4] = [
        (KeyCode::ArrowLeft, Vec2::NEG_X),
        (KeyCode::ArrowRight, Vec2::X),
        (KeyCode::ArrowUp, Vec2::Y),
        (KeyCode::ArrowDown, Vec2::NEG_Y),
    ];
    
    MOVEMENT_KEYS.iter()
        .filter(|(key, _)| input.pressed(*key))
        .map(|(_, dir)| *dir)
        .sum()
}
```

**What's happening here?**

We define a constant array mapping each arrow key to its direction vector. `Vec2::NEG_X` means "negative X direction" (left), `Vec2::X` means "positive X direction" (right), and so on.

Then we iterate through all four keys, filter to only the ones currently pressed, extract their direction vectors, and sum them. If both Up and Right are pressed, we get `Vec2::Y + Vec2::X` = `Vec2 { x: 1.0, y: 1.0 }`.

**What's `_` in `(keys, _)` and in `(_, dir)`?**

The `_` (underscore) is Rust's "I don't care" placeholder in pattern matching. When destructuring tuples, you use `_` to ignore values you don't need:

- In `(key, _)`: We only need the `key` to check if it's pressed, so we ignore the direction with `_`
- In `(_, dir)`: We only need the `dir` (direction vector), so we ignore the key with `_`

This is more readable than naming unused variables like `(key, _unused_dir)` or `(_unused_key, dir)`. Rust's compiler also knows these values are intentionally ignored, so you won't get warnings about unused variables.

### Calculating Movement Speed

Different characters move at different speeds. The Male character might be slower, while the Female character is faster. We also need to support running (holding Shift).

Add this helper function:

```rust
// Append this to src/characters/movement.rs
/// Calculate movement speed based on character config and running state
fn calculate_movement_speed(character: &CharacterEntry, is_running: bool) -> f32 {
    if is_running {
        character.base_move_speed * character.run_speed_multiplier
    } else {
        character.base_move_speed
    }
}
```

This reads the character's `base_move_speed` from our data file and multiplies it by `run_speed_multiplier` if the player is holding Shift. All the speed values are data-driven—no hardcoded constants!

### The Player Marker

We need a way to identify which entity is the player. We'll use a simple marker component:

```rust
// Append this to src/characters/movement.rs
/// Marker component for the player entity
#[derive(Component)]
pub struct Player;
```

This component has no data, it's just a tag. When we spawn the player entity, we'll attach this component. Then our movement system can query for entities with `Player` to find the player. We have already studied this in the first chapter.

### The Movement System

Now we tie it all together. This system runs every frame, reads input, calculates movement, and updates the animation state:

```rust
// Append this to src/characters/movement.rs
/// Handle player movement input and update transform/animation
pub fn move_player(
    input: Res<ButtonInput<KeyCode>>,
    time: Res<Time>,
    mut query: Query<(
        &mut Transform, 
        &mut AnimationController,
        &mut AnimationState,
        &CharacterEntry,
    ), With<Player>>,
) {
    let Ok((mut transform, mut animated, mut state, character)) = query.single_mut() else {
        return;
    };
    
    let direction = read_movement_input(&input);
    
    // Check for jump input (space key)
    if input.just_pressed(KeyCode::Space) {
        state.is_jumping = true;
        animated.current_animation = AnimationType::Jump;
    }
    
    // Check if running
    let is_running = input.pressed(KeyCode::ShiftLeft) || input.pressed(KeyCode::ShiftRight);
    
    // Handle movement
    if direction != Vec2::ZERO {
        let move_speed = calculate_movement_speed(character, is_running);
        let delta = direction.normalize() * move_speed * time.delta_secs();
        transform.translation += delta.extend(0.0);
        
        animated.facing = Facing::from_direction(direction);
        
        // Only update animation if not jumping
        if !state.is_jumping {
            state.is_moving = true;
            animated.current_animation = if is_running {
                AnimationType::Run
            } else {
                AnimationType::Walk
            };
        }
    } else if !state.is_jumping {
        state.is_moving = false;
        animated.current_animation = AnimationType::Walk;
    }
}
```

**Breaking it down:**

1. **Query for the player**: `With<Player>` filters to only entities with the `Player` component. `single_mut()` gets the one player entity.
2. **Read input**: Get the direction vector from arrow keys.
3. **Handle jumping**: If Space was just pressed, set `is_jumping` and switch to the Jump animation.
4. **Check for running**: Are either Shift keys pressed?
5. **Move the character**: If there's input, normalize the direction (so diagonal movement isn't faster), multiply by speed and delta time, and update the transform.

```comic
left_guy_smile: Diagonal movement is faster! It's a feature!
right_girl_sad: It's a bug. Normalize your vectors.
```
6. **Update facing**: Use our `Facing::from_direction` helper to determine which way to face.
7. **Update animation**: If not jumping, set the animation to Run or Walk based on whether `Shift` is held.

### Handling Jump Completion

![Sprite Sheets]({{ "/assets/book_assets/chapter3/jump.gif" | relative_url }})


Jump animations are special, they have a beginning and an end. Unlike Walk or Run, which loop forever, Jump plays once and then we need to return to the idle state.

Add this system:

```rust
// Append this to src/characters/movement.rs
/// Monitor jump animation completion and reset state
pub fn update_jump_state(
    mut query: Query<(
        &mut AnimationController,
        &mut AnimationState,
        &AnimationTimer,
        &Sprite,
        &CharacterEntry,
    ), With<Player>>,
) {
    for (mut animated, mut state, timer, sprite, config) in query.iter_mut() {
        if !state.is_jumping {
            continue;
        }
        
        let Some(atlas) = sprite.texture_atlas.as_ref() else {
            continue;
        };
        
        let Some(clip) = animated.get_clip(config) else {
            continue;
        };
        
        // Check if jump animation has completed
        if clip.is_complete(atlas.index, timer.just_finished()) {
            state.is_jumping = false;
            animated.current_animation = AnimationType::Walk;
        }
    }
}
```

This system checks if the jump animation has reached its last frame and the timer has finished (using the `is_complete` method we defined earlier in `AnimationClip`). If so, it resets `is_jumping` to false and switches back to the Walk animation. The player can then move normally again.

**What's `.as_ref()`?**

In the animation system, we used `.as_mut()` to get a mutable reference to the texture atlas so we could change the frame index. Here, we only need to *read* the current frame index, not modify it. The `.as_ref()` method converts `Option<TextureAtlas>` into `Option<&TextureAtlas>`, giving us a read-only reference. 

## Spawning Characters

We have animation, movement, and data structures, but no actual character on screen yet! The spawn system is responsible for:

1. Loading the `characters.ron` file
2. Creating the player entity with all necessary components
3. Setting up the texture atlas from the spritesheet
4. Allowing runtime character switching with number keys (1-6)

Create a new file `src/characters/spawn.rs`:

```
characters/
├── config.rs
├── animation.rs
├── movement.rs
├── spawn.rs  <- Create this
```

### Resources for Character Management

We'll need two resources: one to track which character is currently active, and another to hold a reference to our loaded character data file.

Add this to `src/characters/spawn.rs`:

```rust
// src/characters/spawn.rs
use bevy::prelude::*;
use crate::characters::animation::*;
use crate::characters::config::{CharacterEntry, CharactersList};
use crate::characters::movement::Player;

const PLAYER_SCALE: f32 = 0.8;
const PLAYER_Z_POSITION: f32 = 20.0;

#[derive(Resource, Default)]
pub struct CurrentCharacterIndex {
    pub index: usize,
}

#[derive(Resource)]
pub struct CharactersListResource {
    pub handle: Handle<CharactersList>,
}
```

**What are these resources for?**

- `CurrentCharacterIndex`: Tracks which character is currently active (0 = first character, 1 = second, etc.)
- `CharactersListResource`: Stores the handle to our loaded `characters.ron` file. A handle is like a reference to an asset that Bevy is loading or has loaded.

**How to decide between using resources and components?**

Use **Components** for data that belongs to specific entities (like a player's health, position, or animation state). Use **Resources** for global data that isn't tied to any particular entity (like the current level number, game settings, or in our case, which character is active). Think of it this way: if you'd ask "which entity does this belong to?" and the answer is "all of them" or "none of them," it's probably a Resource.

```comic
left_guy_surprised: I made player health a Resource instead of Component.
right_girl_laugh: Ah yes, the communist approach. Everyone shares one health bar.
```

**What's the `Default` macro?**

The `Default` derive macro automatically implements the `Default` trait, which provides a default value for the struct. For `usize`, Rust's default is `0`. When you later in the chapter use `init_resource::<CurrentCharacterIndex>()`, Bevy internally calls `CurrentCharacterIndex::default()`, which creates `CurrentCharacterIndex { index: 0 }`.

This is equivalent to manually writing:
```rust
// Pseudo code warning, don't use
impl Default for CurrentCharacterIndex {
    fn default() -> Self {
        Self { index: 0 }
    }
}
```

But the derive macro does it for us! Different types have different defaults: `bool` → `false`, `String` → `""`, `Option<T>` → `None`, etc.

### Creating the Texture Atlas Layout

Remember how we talked about spritesheets being grids of frames? Bevy doesn't automatically know where one frame ends and another begins. We need to give it instructions: "Each frame is 64×64 pixels, there are 12 columns, and 8 rows." That's what a texture atlas layout does, it's like a map that tells Bevy how to navigate the spritesheet.

This helper function creates that map:

```rust
// Append this to src/characters/spawn.rs
/// Create a texture atlas layout for a character
fn create_character_atlas_layout(
    atlas_layouts: &mut ResMut<Assets<TextureAtlasLayout>>,
    character_entry: &CharacterEntry,
) -> Handle<TextureAtlasLayout> {
    let max_row = character_entry.calculate_max_animation_row();
    
    atlas_layouts.add(TextureAtlasLayout::from_grid(
        UVec2::splat(character_entry.tile_size),
        character_entry.atlas_columns as u32,
        (max_row + 1) as u32,
        None,
        None,
    ))
}
```

**Breaking it down:**

- `calculate_max_animation_row()`: We defined this earlier in `CharacterEntry`. It figures out how many rows the spritesheet needs based on all the animations.
- `UVec2::splat(tile_size)`: Creates a 2D vector where both x and y are the tile size (e.g., 64×64 pixels per frame).
- `from_grid(...)`: Tells Bevy "this texture is a grid with X columns and Y rows, each cell is this size."

### Spawning the Player Entity

Now we spawn the player. But here's the catch: loading files from disk takes time. We can't wait for `characters.ron` to load before creating the player entity, that would freeze the game during startup.

So we use a two stage approach: create a "placeholder" entity immediately, then fill in the details once the data finishes loading. 

**Stage 1: Create the entity**

```rust
// Append this to src/characters/spawn.rs
pub fn spawn_player(
    mut commands: Commands,
    asset_server: Res<AssetServer>,
    mut character_index: ResMut<CurrentCharacterIndex>,
) {
    // Load the characters list
    let characters_list_handle: Handle<CharactersList> = asset_server.load("characters/characters.ron");
    
    // Store the handle in a resource
    commands.insert_resource(CharactersListResource {
        handle: characters_list_handle,
    });
    
    // Initialize with first character
    character_index.index = 0;
    
    // Spawn player entity (will be initialized once asset loads)
    commands.spawn((
        Player,
        Transform::from_translation(Vec3::new(0.0, 0.0, PLAYER_Z_POSITION))
            .with_scale(Vec3::splat(PLAYER_SCALE)),
        Sprite::default(),
    ));
}
```

This system runs once at startup. It loads the `characters.ron` file, stores the handle in a resource, and spawns a player entity with just the `Player` marker, a `Transform`, and an empty `Sprite`. We can't fully initialize it yet because the asset is still loading.

**Why not load everything immediately?**

Asset loading in Bevy is asynchronous. When you call `asset_server.load()`, Bevy starts loading the file in the background. It might take a few frames (or longer for large files). We need to wait until it's ready.

```comic
left_guy_anxious: I spawned him, but he's invisible!
right_girl_smile: Relax. The asset server is still loading his pants.
```

**Stage 2: Initialize once loaded**

```rust
// Append this to src/characters/spawn.rs
pub fn initialize_player_character(
    mut commands: Commands,
    asset_server: Res<AssetServer>,
    mut atlas_layouts: ResMut<Assets<TextureAtlasLayout>>,
    characters_lists: Res<Assets<CharactersList>>,
    character_index: Res<CurrentCharacterIndex>,
    characters_list_res: Option<Res<CharactersListResource>>,
    mut query: Query<Entity, (With<Player>, Without<AnimationController>)>,
) {
    let Some(characters_list_res) = characters_list_res else {
        return;
    };
    
    for entity in query.iter_mut() {
        let Some(characters_list) = characters_lists.get(&characters_list_res.handle) else {
            continue;
        };
        
        if character_index.index >= characters_list.characters.len() {
            continue;
        };
        
        let character_entry = &characters_list.characters[character_index.index];
        
        let texture = asset_server.load(&character_entry.texture_path);
        let layout = create_character_atlas_layout(&mut atlas_layouts, character_entry);
        
        let sprite = Sprite::from_atlas_image(
            texture,
            TextureAtlas {
                layout,
                index: 0,
            },
        );
        
        commands.entity(entity).insert((
            AnimationController::default(),
            AnimationState::default(),
            AnimationTimer(Timer::from_seconds(DEFAULT_ANIMATION_FRAME_TIME, TimerMode::Repeating)),
            character_entry.clone(),
            sprite,
        ));
    }
}
```

Earlier in Stage 1, we started loading `characters.ron` in the background. But we don't know *when* it will finish—could be the next frame, could be 10 frames later.

This system runs every frame, checking: "Is the file loaded yet? Is there a player entity that still needs initialization?" 

The query `(With<Player>, Without<AnimationController>)` finds player entities that exist but haven't been fully set up yet. Once the file loads, `characters_lists.get()` succeeds, and we can finally add all the animation components.

We grab the character data, load its texture, create the atlas layout, and insert all the necessary components.

### Character Switching

One of the coolest features of our data-driven system: switching characters at runtime by pressing number keys!

```rust
// Append this to src/characters/spawn.rs
pub fn switch_character(
    input: Res<ButtonInput<KeyCode>>,
    mut character_index: ResMut<CurrentCharacterIndex>,
    characters_lists: Res<Assets<CharactersList>>,
    characters_list_res: Option<Res<CharactersListResource>>,
    mut query: Query<(
        &mut CharacterEntry,
        &mut Sprite,
    ), With<Player>>,
    mut atlas_layouts: ResMut<Assets<TextureAtlasLayout>>,
    asset_server: Res<AssetServer>,
) {
    // Map digit keys to indices
    const DIGIT_KEYS: [KeyCode; 9] = [
        KeyCode::Digit1, KeyCode::Digit2, KeyCode::Digit3,
        KeyCode::Digit4, KeyCode::Digit5, KeyCode::Digit6,
        KeyCode::Digit7, KeyCode::Digit8, KeyCode::Digit9,
    ];
    
    // Find which digit key was pressed
    let new_index = DIGIT_KEYS.iter()
        .position(|&key| input.just_pressed(key));
    
    let Some(new_index) = new_index else {
        return;
    };
    
    let Some(characters_list_res) = characters_list_res else {
        return;
    };
    
    let Some(characters_list) = characters_lists.get(&characters_list_res.handle) else {
        return;
    };
    
    if new_index >= characters_list.characters.len() {
        return;
    }
    
    // Update character index
    character_index.index = new_index;
    
    // Update player entity
    let Ok((mut current_entry, mut sprite)) = query.single_mut() else {
        return;
    };
    
    let character_entry = &characters_list.characters[new_index];
    
    // Update character entry
    *current_entry = character_entry.clone();
    
    // Update sprite with new texture
    let texture = asset_server.load(&character_entry.texture_path);
    let layout = create_character_atlas_layout(&mut atlas_layouts, character_entry);
    
    *sprite = Sprite::from_atlas_image(
        texture,
        TextureAtlas {
            layout,
            index: 0,
        },
    );
}
```

**How it works:**

1. **Detect key press**: Check if any digit key (1-9) was just pressed.
2. **Validate index**: Make sure the index is within bounds (we have 6 characters, so keys 1-6 work).
3. **Update resources**: Set `character_index.index` to the new value.
4. **Swap the character**: Replace the `CharacterEntry` component with the new character's data.
5. **Update the sprite**: Load the new character's texture and create a new atlas layout.

This is the payoff of our data-driven design! Remember how in Chapter 1, adding a second character would have meant duplicating all the animation and movement code? Here, we just swap out the data. The animation system doesn't care if it's animating a Male character or a Crimson Count, it just reads the `CharacterEntry` component and does its job. Same with movement: it reads `base_move_speed` and `run_speed_multiplier` from the new character's data. No code changes needed.

**How does `.position()` work?**

This iterator method searches through the array and returns the index of the first item where the condition is true. The closure `|&key| input.just_pressed(key)` checks each key: "was this key just pressed?" If the player presses `Digit3`, `.position()` returns `Some(2)` (because `Digit3` is at index 2 in the array). If no digit key is pressed, it returns `None`. It's similar to the filter operation.

**What's the `*` in `*current_entry`?**

The `*` is the dereference operator. `current_entry` is a mutable reference (`&mut CharacterEntry`), not the actual data. To modify the data it points to, we need to dereference it with `*`. Think of it like this: `current_entry` is a pointer to a box, `*current_entry` is the contents of the box. We're replacing the contents, not the pointer.

```comic
left_guy_smile: I can swap 6 characters with zero code duplication!
right_girl_surprised: Impossible. Next you'll tell me you actually read the Bevy docs.
```

## Bringing It All Together

We've built all the pieces: animation, movement, spawning, and character switching. Now we need to package them into a plugin and integrate it into our game.

### Adding the RON Asset Loader

First, we need to add dependencies that let Bevy load `.ron` files and serialize/deserialize our data structures. Open `Cargo.toml` and add these to your `[dependencies]` section:

```toml
bevy_common_assets = { version = "0.15.0-rc.1", features = ["ron"] }
serde = { version = "1.0", features = ["derive"] }
```

The `bevy_common_assets` crate provides asset loaders for common file formats. We're using the `ron` feature to load our `characters.ron` file. The `serde` crate with the `derive` feature allows us to use `#[derive(Serialize, Deserialize)]` on our structs, which we used in `config.rs` and `animation.rs`.

### Creating the Characters Plugin

Create `src/characters/mod.rs`:

```rust
// src/characters/mod.rs
pub mod animation;
pub mod config;
pub mod movement;
pub mod spawn;

use bevy::prelude::*;
use bevy_common_assets::ron::RonAssetPlugin;
use config::CharactersList;

pub struct CharactersPlugin;

impl Plugin for CharactersPlugin {
    fn build(&self, app: &mut App) {
        app.add_plugins(RonAssetPlugin::<CharactersList>::new(&["characters.ron"]))
            .init_resource::<spawn::CurrentCharacterIndex>()
            .add_systems(Startup, spawn::spawn_player)
            .add_systems(Update, (
                spawn::initialize_player_character,
                spawn::switch_character,
                movement::move_player,
                movement::update_jump_state,
                animation::animate_characters,
                animation::update_animation_flags,
            ));
    }
}
```

**Breaking it down:**

- **Module declarations**: `pub mod animation;` etc. make our submodules accessible.
- **RonAssetPlugin**: Registers the `.ron` file loader for our `CharactersList` type.
- **init_resource**: Creates the `CurrentCharacterIndex` resource with its default value (0).
- **Startup systems**: `spawn_player` runs once at game start.
- **Update systems**: All other systems run every frame.

**Why are we keeping `initialize_player_character` in the Update instead of Startup? Does it initialize player character every frame?**

Good catch! The system *does* run every frame, but it doesn't initialize the player every frame. Look at the query: `Query<Entity, (With<Player>, Without<AnimationController>)>`. This only matches player entities that *don't have* an `AnimationController` yet.

Once we add the `AnimationController` component (which happens inside the system), the entity no longer matches the query, so the system does nothing on subsequent frames. It's a self-terminating system—it runs until it finds and initializes uninitialized players, then effectively becomes a no-op.

We can't put it in Startup because the `characters.ron` file might not be loaded yet when Startup runs. By putting it in Update, it keeps checking every frame: "Is the file loaded? Is there an uninitialized player?" Once both conditions are true, it initializes the player and then stops doing anything.

### Plugging Into the Game

Now we connect our plugin to the main game. Open `src/main.rs` and add the module declaration at the top:

```rust
// Update line in src/main.rs
mod map;
mod characters;  // Add this line
```

Then add the plugin to your app:

```rust
// Update line in src/main.rs
fn main() {
    let map_size = map_pixel_dimensions();

    App::new()
        .insert_resource(ClearColor(Color::WHITE))
        .add_plugins(
            DefaultPlugins
                .set(AssetPlugin {
                    file_path: "src/assets".into(),
                    ..default()
                })
                .set(WindowPlugin {
                    primary_window: Some(Window {
                        resolution: WindowResolution::new(map_size.x as u32, map_size.y as u32),
                        resizable: false,
                        ..default()
                    }),
                    ..default()
                })
                .set(ImagePlugin::default_nearest()),
        )
        .add_plugins(ProcGenSimplePlugin::<Cartesian3D, Sprite>::default())
        .add_plugins(characters::CharactersPlugin)  // Add this line
        .add_systems(Startup, (setup_camera, setup_generator))
        .run();
}
```

That's it! Run your game with `cargo run`, and you should see your character on screen. Press the arrow keys to move, hold Shift to run, press Space to jump, and press number keys 1-6 to switch characters!

![Animation System Demo]({{ "/assets/book_assets/chapter3/chapter3.gif" | relative_url }})


<div style="margin: 20px 0; padding: 15px; background-color: #d4edda; border-radius: 8px; border-left: 4px solid #28a745;">
<strong>Next up: Chapter 4!</strong> <br> <a href="/posts/bevy-rust-game-development-chapter-4/">Chapter 4: Let There Be Collisions</a> 
<br><br>
<a href="https://discord.com/invite/cD9qEsSjUH">Join our community</a> to get notified when new chapters drop and share your amazing creations with fellow developers.
<br><br>
<strong>Let's stay connected! Here are some ways</strong>
<ul>
<li>Follow the project on <a href="https://github.com/jamesfebin/ImpatientProgrammerBevyRust">GitHub</a></li>
<li>Join the discussion on <a href="https://www.reddit.com/r/bevy/comments/1p1n2x6/the_impatient_programmers_guide_to_bevy_and_rust/">Reddit</a></li>
<li>Connect with me on <a href="https://www.linkedin.com/in/febinjohnjames">LinkedIn</a> and <a href="https://x.com/heyfebin">X/Twitter</a></li>
</ul>
</div>

