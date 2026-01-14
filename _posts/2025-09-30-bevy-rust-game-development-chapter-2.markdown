---
layout: post
title: "The Impatient Programmer's Guide to Bevy and Rust: Chapter 2 - Let There Be a World"
date: 2025-10-11 10:00:00 +0000
category: rust
excerpt: "Continue building your video game with Bevy and Rust. This second chapter covers procedural world generation using Wave Function Collapse, creating layered terrain with tiles, and building a complete game world with dirt, grass, water, and props."
image: /assets/book_assets/chapter2/ch2.gif
og_image: /assets/book_assets/chapter2/props.png
---

<style>
.tile-image {
  margin: 0 !important;
  object-fit: none !important;
  cursor: default !important;
  pointer-events: none !important;
}
</style>

By the end of this tutorial, you'll build a procedurally generated game world with layered terrain, water bodies, and props.

> **Prerequisites**: This is Chapter 2 of our Bevy tutorial series. [Join our community](https://discord.com/invite/cD9qEsSjUH) for updates on new releases. Before starting, complete [Chapter 1: Let There Be a Player](/posts/bevy-rust-game-development-chapter-1/), or clone the Chapter 1 code from [this repository](https://github.com/jamesfebin/ImpatientProgrammerBevyRust) to follow along.

![Player Movement Demo]({{ "/assets/book_assets/chapter2/ch2.gif" | relative_url }})


<div style="margin: 20px 0; padding: 15px; background-color: #e3f2fd; border-radius: 8px; border-left: 4px solid #1976d2;">
<strong>Before We Begin:</strong> <em style="font-size: 14px;">I'm constantly working to improve this tutorial and make your learning journey enjoyable. Your feedback matters - share your frustrations, questions, or suggestions on <a href="https://www.reddit.com/r/bevy/comments/1o3y2hr/the_impatient_programmers_guide_to_bevy_and_rust/" target="_blank">Reddit</a>/<a target="_blank" href="https://discord.com/invite/cD9qEsSjUH">Discord</a>/<a href="https://www.linkedin.com/posts/febinjohnjames_chapter-2-let-there-be-a-world-continuing-activity-7382797407824039936-WXFD?utm_source=share&utm_medium=member_desktop&rcm=ACoAAAlx1JIBRLKRFr1OsUTf1LYBNPYbgdfxjbc" target="_blank">LinkedIn</a>. Loved it? Let me know what worked well for you! Together, we'll make game development with Rust and Bevy more accessible for everyone.</em>
</div>

## Procedural Generation

I respect artists who hand craft tiles to build game worlds. But I belong to the impatient/lazy species. 

I went on an exploration and came across procedural generation. 

Little did I know the complexities involved. I was on the verge of giving up, however because of the comments and messages from readers of previous chapter, I kept going. And the enlightenment came three days ago, all the pieces fit together.

Basically it's about automatically fitting things together like a jigsaw puzzle. To solve this problem, let's again think in systems. 

**What do we need to generate the game world procedurally?**
1. Tileset.
2. Sockets for tiles because only compatible tiles should fit.
3. Compatibility rules.
4. Magic algorithm that uses these components to generate a coherent world.

**How does this magic algorithm work?**

That "magic algorithm" has a name: Wave Function Collapse (WFC). The easiest way to see it is with a tiny Sudoku. Same idea: pick the cell with the fewest valid options, place a value, update neighbors, and repeat. If a choice leads to a dead end, undo that guess and try the next option.


**Small 4×4 Sudoku**

Let's solve this step by step, focusing on the most constrained cells first. 

<link href="https://fonts.googleapis.com/css2?family=VT323&display=swap" rel="stylesheet">

<div style="margin: 20px 0; padding: 15px; background-color: #d1ecf1; border-radius: 8px; border-left: 4px solid #17a2b8;">
<strong>Initial Puzzle:</strong> We need to fill in the empty cells (marked with dots) following Sudoku rules.
</div>

<div class="columns is-mobile is-centered">
<div class="column is-narrow">
<table style="border-collapse: separate; border-spacing: 2px; font-family: 'VT323', monospace; font-size: 32px; border-radius: 8px; overflow: hidden; border: none; display: inline-block; margin-bottom: 20px;">
<tr>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7 !important; border-radius: 8px !important;">?</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #e3f2fd !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">2</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
</tr>
<tr>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #e3f2fd !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #e3f2fd !important; border-radius: 8px !important;">3</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
</tr>
<tr>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">1</td>
</tr>
<tr>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">4</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
</tr>
</table>
</div>
</div>

<div style="margin: 20px 0; padding: 15px; background-color: #fff3cd; border-radius: 8px; border-left: 4px solid #ffc107;">
<strong>Step 1 — Finding the most constrained cell:</strong><br>
Let's analyze the top-left 2×2 box:
<ul>
<li>Row 1 already has: 2</li>
<li>Column 1 already has: 4</li>
<li>Top-left box already has: 3</li>
<li>Available numbers: 1, 2, 3, 4</li>
<li>Eliminating: 2 (in row), 4 (in column), 3 (in box)</li>
<li><strong>Only 1 remains!</strong></li>
</ul>
</div>

<div class="columns is-mobile is-centered">
<div class="column is-narrow">
<table style="border-collapse: separate; border-spacing: 2px; font-family: 'VT323', monospace; font-size: 32px; border-radius: 8px; overflow: hidden; border: none; display: inline-block; margin-bottom: 20px;">
<tr>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9 !important; border-radius: 8px !important; font-weight: bold;">1</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #e3f2fd !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">2</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
</tr>
<tr>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #e3f2fd !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #e3f2fd !important; border-radius: 8px !important;">3</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
</tr>
<tr>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">1</td>
</tr>
<tr>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">4</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
</tr>
</table>
</div>
</div>

<div style="margin: 20px 0; padding: 15px; background-color: #d1ecf1; border-radius: 8px; border-left: 4px solid #17a2b8;">
<strong>Propagation Effect:</strong> Now that we placed 1, we can eliminate 1 from:
<ul>
<li>Row 1: ✓ (already done)</li>
<li>Column 1: ✓ (already done)</li>
<li>Top-left 2×2 box: ✓ (already done)</li>
</ul>
This makes other cells more constrained!
</div>

<div style="margin: 20px 0; padding: 15px; background-color: #fff3cd; border-radius: 8px; border-left: 4px solid #ffc107;">
<strong>Step 2 — Next most constrained cell:</strong><br>
Now let's find the next cell with the fewest options. 
</div>

<div class="columns is-mobile is-centered">
<div class="column is-narrow">
<table style="border-collapse: separate; border-spacing: 2px; font-family: 'VT323', monospace; font-size: 32px; border-radius: 8px; overflow: hidden; border: none; display: inline-block; margin-bottom: 20px;">
<tr>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9 !important; border-radius: 8px !important;">1</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7 !important; border-radius: 8px !important;">?</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">2</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
</tr>
<tr>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #e3f2fd !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #e3f2fd !important; border-radius: 8px !important;">3</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
</tr>
<tr>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">1</td>
</tr>
<tr>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">4</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
</tr>
</table>
</div>
</div>

<div style="margin: 20px 0; padding: 15px; background-color: #d1ecf1; border-radius: 8px; border-left: 4px solid #17a2b8;">
<strong>Analysis for the position:</strong>
<ul>
<li>Row 1 already has: 1, 2</li>
<li>Column 2 already has: 3</li>
<li>Top-left box already has: 1, 3</li>
<li>Available numbers: 1, 2, 3, 4</li>
<li>Eliminating: 1 (in row), 2 (in row), 3 (in column and box)</li>
<li><strong>Only 4 remains!</strong></li>
</ul>
</div>

<div class="columns is-mobile is-centered">
<div class="column is-narrow">
<table style="border-collapse: separate; border-spacing: 2px; font-family: 'VT323', monospace; font-size: 32px; border-radius: 8px; overflow: hidden; border: none; display: inline-block; margin-bottom: 20px;">
<tr>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9 !important; border-radius: 8px !important;">1</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9 !important; border-radius: 8px !important; font-weight: bold;">4</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">2</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
</tr>
<tr>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #e3f2fd !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #e3f2fd !important; border-radius: 8px !important;">3</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
</tr>
<tr>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">1</td>
</tr>
<tr>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">4</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
<td style="padding: 12px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #f5f5f5 !important; border-radius: 8px !important;">.</td>
</tr>
</table>
</div>
</div>

<div style="margin: 20px 0; padding: 15px; background-color: #d4edda; border-radius: 8px; border-left: 4px solid #28a745;">
<strong>Key Insight</strong> <br> This is the essence of constraint propagation! Each placement immediately reduces the options for neighboring cells, making the puzzle progressively easier to solve. 
<br><br>
We continue this process: pick the most constrained cell → place the only possible value → propagate constraints → repeat. 
<br><br>
If any cell ends up with zero possibilities, we've hit a contradiction—in Sudoku, you backtrack and try a different value.
</div>

```d2
direction: right

Pick: {
  label: |md
    **Pick cell**
    - fewest possiblities
  |
}

Filter: {
  label: |md
    **Filter**
    - apply rules
  |
}

Place: {
  label: |md
    **Place**
    - choose one option
  |
}

Propagate: {
  label: |md
    **Propagate**
    - update neighbors
    - detect contradictions
  |
}

Backtrack: {
  label: |md
    **Backtrack**
    - undo last guess
    - try next
  |
}

Pick -> Filter -> Place -> Propagate
Propagate -> Pick: repeat
Propagate -> Backtrack: if contradiction
Backtrack -> Pick
```



**For our tile-based world:** Imagine each grid cell as a Sudoku cell, but instead of numbers, we're placing tiles. 

Then we craft rules for valid connections:
- **Rule 1**: Water center tiles connect to other water tiles on all sides
- **Rule 2**: Water edge tiles have two types of sides - water-facing sides connect to other water tiles, land-facing sides connect to the shore

The algorithm uses these rules to ensure tiles fit together properly, creating coherent water bodies with natural-looking shorelines.

Let's see this in action:

<div class="columns is-mobile is-centered">
<div class="column is-narrow">
<table style="border-collapse: separate; border-spacing: 2px; font-family: 'VT323', monospace; font-size: 24px; border-radius: 8px; overflow: hidden; border: none; display: inline-block; margin-bottom: 20px;">
<tr>
<td style="padding: 4px; text-align: center; width: 50px; height: 50px; border: none !important; background-color: #f5f5f5; border-radius: 6px !important;"><img src="/assets/book_assets/tile_layers/water.png" alt="Water center" class="tile-image" style="width: 50px; height: 42px; display: block;"></td>
<td style="padding: 4px; text-align: center; width: 50px; height: 50px; border: none !important; background-color: #f5f5f5; border-radius: 6px !important;"><img src="/assets/book_assets/tile_layers/water_side_t.png" alt="Water top" class="tile-image" style="width: 50px; height: 42px; display: block;"></td>
<td style="padding: 4px; text-align: center; width: 50px; height: 50px; border: none !important; background-color: #f5f5f5; border-radius: 6px !important;"><img src="/assets/book_assets/tile_layers/water_side_b.png" alt="Water bottom" class="tile-image" style="width: 50px; height: 42px; display: block;"></td>
<td style="padding: 4px; text-align: center; width: 50px; height: 50px; border: none !important; background-color: #f5f5f5; border-radius: 6px !important;"><img src="/assets/book_assets/tile_layers/water_side_l.png" alt="Water left" class="tile-image" style="width: 50px; height: 42px; display: block;"></td>
<td style="padding: 4px; text-align: center; width: 50px; height: 50px; border: none !important; background-color: #f5f5f5; border-radius: 6px !important;"><img src="/assets/book_assets/tile_layers/water_side_r.png" alt="Water right" class="tile-image" style="width: 50px; height: 42px; display: block;"></td>
</tr>
<tr>
<td style="padding: 4px; text-align: center; width: 50px; height: 50px; border: none !important; background-color: #f5f5f5; border-radius: 6px !important;"><img src="/assets/book_assets/tile_layers/water_corner_in_tl.png" alt="Water corner in TL" class="tile-image" style="width: 50px; height: 42px; display: block;"></td>
<td style="padding: 4px; text-align: center; width: 50px; height: 50px; border: none !important; background-color: #f5f5f5; border-radius: 6px !important;"><img src="/assets/book_assets/tile_layers/water_corner_in_tr.png" alt="Water corner in TR" class="tile-image" style="width: 50px; height: 42px; display: block;"></td>
<td style="padding: 4px; text-align: center; width: 50px; height: 50px; border: none !important; background-color: #f5f5f5; border-radius: 6px !important;"><img src="/assets/book_assets/tile_layers/water_corner_in_bl.png" alt="Water corner in BL" class="tile-image" style="width: 50px; height: 42px; display: block;"></td>
<td style="padding: 4px; text-align: center; width: 50px; height: 50px; border: none !important; background-color: #f5f5f5; border-radius: 6px !important;"><img src="/assets/book_assets/tile_layers/water_corner_in_br.png" alt="Water corner in BR" class="tile-image" style="width: 50px; height: 42px; display: block;"></td>
<td style="padding: 4px; text-align: center; width: 50px; height: 50px; border: none !important; background-color: #f5f5f5; border-radius: 6px !important;"><img src="/assets/book_assets/tile_layers/water_corner_out_tl.png" alt="Water corner out TL" class="tile-image" style="width: 50px; height: 42px; display: block;"></td>
</tr>
<tr>
<td style="padding: 4px; text-align: center; width: 50px; height: 50px; border: none !important; background-color: #f5f5f5; border-radius: 6px !important;"><img src="/assets/book_assets/tile_layers/water_corner_out_tr.png" alt="Water corner out TR" class="tile-image" style="width: 50px; height: 42px; display: block;"></td>
<td style="padding: 4px; text-align: center; width: 50px; height: 50px; border: none !important; background-color: #f5f5f5; border-radius: 6px !important;"><img src="/assets/book_assets/tile_layers/water_corner_out_bl.png" alt="Water corner out BL" class="tile-image" style="width: 50px; height: 42px; display: block;"></td>
<td style="padding: 4px; text-align: center; width: 50px; height: 50px; border: none !important; background-color: #f5f5f5; border-radius: 6px !important;"><img src="/assets/book_assets/tile_layers/water_corner_out_br.png" alt="Water corner out BR" class="tile-image" style="width: 50px; height: 42px; display: block;"></td>

</tr>
</table>
</div>
</div>

**Step 1 - Initial Grid**

<div class="columns is-mobile is-centered">
<div class="column is-narrow">
<table style="border-collapse: separate; border-spacing: 2px; font-family: 'VT323', monospace; font-size: 32px; border-radius: 8px; overflow: hidden; border: none; display: inline-block; margin-bottom: 20px;">
<tr>
<td style="padding: 8px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
<td style="padding: 8px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
<td style="padding: 8px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
<td style="padding: 8px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
</tr>
<tr>
<td style="padding: 8px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
<td style="padding: 8px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
<td style="padding: 8px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
<td style="padding: 8px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
</tr>
</table>
</div>
</div>

We start with an empty grid where every cell can potentially hold any tile. The `?` symbols represent the "superposition" - each cell contains all possible tiles until we begin constraining them through the algorithm.

**Step 2 - First Placement**

<div class="columns is-mobile is-centered">
<div class="column is-narrow">
<table style="border-collapse: separate; border-spacing: 2px; font-family: 'VT323', monospace; font-size: 32px; border-radius: 8px; overflow: hidden; border: none; display: inline-block; margin-bottom: 20px;">
<tr>
<td style="padding: 8px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
<td style="padding: 8px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
<td style="padding: 8px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
<td style="padding: 8px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
</tr>
<tr>
<td style="padding: 8px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important;"><img src="/assets/book_assets/tile_layers/water.png" alt="Water center" class="tile-image" style="width: 60px; height: 50px; display: block;"></td>
<td style="padding: 8px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
<td style="padding: 8px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
</tr>
</table>
</div>
</div>

The algorithm starts by placing the initial water center tile. Following **Rule 1**, this center tile needs other water tiles on all sides. This immediately constrains the neighboring cells - they must be water tiles that can connect to the center.

**Step 3 - Propagate Constraints**

<div class="columns is-mobile is-centered">
<div class="column is-narrow">
<table style="border-collapse: separate; border-spacing: 2px; font-family: 'VT323', monospace; font-size: 32px; border-radius: 8px; overflow: hidden; border: none; display: inline-block; margin-bottom: 20px;">
<tr>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important;"><img src="/assets/book_assets/tile_layers/water_side_t.png" alt="Water edge" class="tile-image" style="width: 60px; height: 50px; display: block;"></td>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important;"><img src="/assets/book_assets/tile_layers/water_side_t.png" alt="Water edge" class="tile-image" style="width: 60px; height: 50px; display: block;"></td>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
</tr>
<tr>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important;"><img src="/assets/book_assets/tile_layers/water.png" alt="Water center" class="tile-image" style="width: 60px; height: 50px; display: block;"></td>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important;"><img src="/assets/book_assets/tile_layers/water.png" alt="Water center" class="tile-image" style="width: 60px; height: 50px; display: block;"></td>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #ffeaa7; border-radius: 8px !important;">?</td>
</tr>
</table>
</div>
</div>

Constraint propagation kicks in! The algorithm expands the water area by placing more center tiles. The edge tiles at the top follow **Rule 2** - their bottom sides (water-facing) connect to the center tiles, while their top sides (land-facing) will connect to the shore. 

**Step 4 - Final Result**

<div class="columns is-mobile is-centered">
<div class="column is-narrow">
<table style="border-collapse: separate; border-spacing: 2px; font-family: 'VT323', monospace; font-size: 32px; border-radius: 8px; overflow: hidden; border: none; display: inline-block; margin-bottom: 20px;">
<tr>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important;"><img src="/assets/book_assets/tile_layers/water_corner_out_tl.png" alt="Water" class="tile-image" style="width: 60px; height: 50px; display: block;"></td>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important;"><img src="/assets/book_assets/tile_layers/water_side_t.png" alt="Water center" class="tile-image" style="width: 60px; height: 50px; display: block;"></td>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important;"><img src="/assets/book_assets/tile_layers/water_side_t.png" alt="Water center" class="tile-image" style="width: 60px; height: 50px; display: block;"></td>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important;"><img src="/assets/book_assets/tile_layers/water_corner_out_tr.png" alt="Water" class="tile-image" style="width: 60px; height: 50px; display: block;"></td>
</tr>
<tr>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important;"><img src="/assets/book_assets/tile_layers/water_side_l.png" alt="Water center" class="tile-image" style="width: 60px; height: 50px; display: block;"></td>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important;"><img src="/assets/book_assets/tile_layers/water.png" alt="Water center" class="tile-image" style="width: 60px; height: 50px; display: block;"></td>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important;"><img src="/assets/book_assets/tile_layers/water.png" alt="Water center" class="tile-image" style="width: 60px; height: 50px; display: block;"></td>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important;"><img src="/assets/book_assets/tile_layers/water_side_r.png" alt="Water center" class="tile-image" style="width: 60px; height: 50px; display: block;"></td>
</tr>
</table>
</div>
</div>

The algorithm completes by filling the edges with appropriate boundary tiles. Notice how our rules create perfect connections - center tiles (**Rule 1**) have water on all sides, while edge and corner tiles (**Rule 2**) have water-facing sides connecting inward and land-facing sides connecting to the shore, creating a coherent geography.

This demonstrates the core Wave Function Collapse algorithm in action:
1. **Find the most constrained cell** - the one with the fewest valid tiles that could fit
2. **Place a tile** whose sockets are compatible with its neighbors  
3. **Propagate constraints** - this placement immediately reduces the valid options for surrounding cells
4. **Repeat** until the grid is complete

When we hit a dead end (no valid tiles for a cell), our implementation takes a simpler approach than Sudoku: instead of backtracking through previous choices, we restart with a fresh random seed (up to a retry limit) and run the entire process again until we generate a valid map.

**What do you mean by fresh random seed?**

A "random seed" is a starting number that controls which "random" sequence the algorithm will follow. Same seed = same tile placement order every time. When we hit a dead end, instead of backtracking, we generate a new random seed and start over—this gives us a completely different sequence of tile choices to try.

**Can configuring this randomness help us customize maps?**

Yes! The algorithm's randomness comes from the order in which it picks cells and tiles, and we can control this to influence the final result. By adjusting the random seed or the selection strategy, we can:

- **Bias toward certain patterns** - Weight certain tiles more heavily to create specific landscape types.
- **Control size and complexity** - Influence whether we get small ponds or large lakes.
- **Create predictable variations** - Use the same seed for consistent results, or different seeds for variety.

The same tileset can generate endless variations of coherent landscapes, from simple ponds to complex branching river systems, all by tweaking the randomness probability configuration.

<div style="margin: 20px 0; padding: 15px; background-color: #fff3cd; border-radius: 8px; border-left: 4px solid #ffc107;">
While Wave Function Collapse is powerful, it has its limitations.
<ul>
<li><strong>No large-scale structure control</strong> - WFC focuses on tile compatibility, so it won't automatically create big patterns like "one large lake" or "mountain ranges".</li>
<li><strong>Can get stuck</strong> - Complex rules might lead to impossible situations where no valid tiles remain, requiring restarts.</li>
<li><strong>Performance depends on complexity</strong> - More tile types and stricter rules increase computation time and failure rates.</li>
<li><strong>Requires careful rule design</strong> - Poorly designed compatibility rules can lead to unrealistic or broken landscapes.</li>
</ul>
We'll address these limitations in a later chapter. For now, we'll focus on building a functional section of our game world that will become the foundation for building larger game worlds.
</div>

## From Theory to Implementation

Now that we understand **how** Wave Function Collapse works—the constraint propagation, socket compatibility, and tile placement logic. It's time to transform this knowledge into actual running code.


**The reality of implementation:**

Building a WFC algorithm from scratch is complex. You'd need to implement:
- Constraint propagation across the entire grid
- Backtracking when hitting dead ends
- Efficient data structures for tracking possibilities
- Grid coordinate management
- Random selection with proper probability weights

That's a lot of algorithmic complexity before we even get to the game-specific parts, like sprites, rules, and world design.

**Our approach:**

Instead of reinventing the wheel, we'll use a library that handles the WFC algorithm internals. This lets us focus on what makes our game unique: the tiles, the rules, the world aesthetics. We define **what** we want; the library figures out **how** to achieve it.


## Setting Up Our Toolkit

Let's add the procedural generation library to our project. We'll be using the `bevy_procedural_tilemaps` [crate](https://crates.io/crates/bevy_procedural_tilemaps), which I built by forking `ghx_proc_gen` [library](https://crates.io/crates/ghx_proc_gen). I created this fork primarily to ensure compatibility with latest Bevy and to simplify this tutorial. 

If you need advanced features, check out the original `ghx_proc_gen` [crate](https://crates.io/crates/ghx_proc_gen) by Guillaume Henaux, which includes 3D capabilities and debugging tools.

Hope you are following the code from first chapter. Here's the [source code](https://github.com/jamesfebin/ImpatientProgrammerBevyRust). 

Update your `Cargo.toml` with the bevy_procedural_tilemaps crate. 

```rust
[package]
name = "bevy_game"
version = "0.1.0"
edition = "2024" 

[dependencies]
bevy = "0.18" // Line update alert 
bevy_procedural_tilemaps = "0.2.0" // Line update alert
```

## Bevy Procedural Tilemaps

The `bevy_procedural_tilemaps` library handles the complex logic of generating coherent, rule-based worlds. 

**What the library handles**

The library takes care of the **algorithmic complexity** of procedural generation:

- **Rule Processing**: Converts our game rules into the library's internal format
- **Generator Creation**: Builds the procedural generation engine with our configuration
- **Constraint Solving**: Figures out which tiles can go where based on rules
- **Grid Management**: Handles the 2D grid system and coordinate transformations
- **Entity Spawning**: Creates Bevy entities and positions them correctly

**What we need to provide**

We need to give the library the **game-specific information** it needs:

- **Sprite Definitions**: What sprites to use for each tile type
- **Compatibility Rules**: Which tiles can be placed next to each other
- **Generation Configuration**: The patterns and constraints for our specific game world
- **Asset Data**: Sprite information, positioning, and custom components


```d2

direction: right

Sprite Atlas: {
  shape: rectangle
}

Asset Definitions: {
  shape: rectangle
}

Compatibility Rules: {
  shape: rectangle
}

Model Organizer: {
  shape: rectangle
}

Generation Config: {
  shape: rectangle
}

Library Interface: {
  shape: rectangle
}

Sprite Atlas -> Asset Definitions: "Sprite names"
Asset Definitions -> Model Organizer: "What to spawn"
Compatibility Rules -> Model Organizer: "What can go where"
Model Organizer -> Library Interface: "Organized models"
Generation Config -> Library Interface: "Generation settings"
```

Now that we understand how the procedural generation system works, let's build our map module.


<div style="margin: 20px 0; padding: 15px; background-color: #fff3cd; border-radius: 8px; border-left: 4px solid #ffc107;">
<strong>Important Note</strong> <br> <br> Unlike Chapter 1 where you could see immediate results, procedural generation requires doing some ground work. You'll create asset loaders, tile spawning infrastructure before seeing your generated world.  <br> <br>

The payoff comes when you finish the grass layer, at that point, you've learned the complete pattern. Adding water and props becomes straightforward repetition with different sprites and connection rules. Along the way, you'll understand Rust concepts (lifetimes, trait bounds, closures) that apply to any Rust project.
  <br> <br>

<b>Don't give up if concepts feel unclear on first read. That's normal with procedural generation. Revisit confusing sections, experiment with the code, and understanding will emerge. Mastery comes through tinkering, not perfect comprehension on the first pass.</b>
</div>


## The Map Module

We'll create a dedicated `map` folder inside the `src` folder to house all our world generation logic.

**Why create a separate folder for map generation?**

The map system requires multiple components working together. World generation involves:

- **Asset management** - Loading and organizing hundreds of tile images.
- **Rule definitions** - Compatibility rules between different terrain types.
- **Grid setup** - Configuring map dimensions and coordinate systems.

Trying to fit all this logic into a single file would create a large file that can become difficult to navigate.

```
src/
├── main.rs
├── player.rs
└── map/
    ├── mod.rs       
    ├── assets.rs       
```

**What's `mod.rs`**

The `mod.rs` file is Rust's way of declaring what modules exist in a folder. It's like the "table of contents" for our map module. Add the following line to your `mod.rs`.

```rust
// src/map/mod.rs
pub mod assets;   // Exposes assets.rs as a module
```

**Why `mod.rs` specifically?**

It's Rust convention, when you create a folder, Rust looks for `mod.rs` to understand the module structure.

### Creating SpawnableAsset

Let's start by creating our `assets.rs` file inside the `map` folder. This will be the foundation that defines how we spawn sprites in our world.

The `bevy_procedural_tilemaps` library needs to know **what to actually place** at each generated location. 

It requires the following details:
1. Which sprite to use from our tilemap atlas?
2. Where exactly to position it?
3. What components to add (collision, physics, etc.)?

The library expects us to provide this information in a very specific format. And doing 
this for every single tile type in your game - grass, dirt, trees, rocks, water, etc will result in redundant code.

This is where `SpawnableAsset` comes in. It's our **abstraction layer** to help you avoid unnecessary boilerplate. 

```rust
// src/map/assets.rs

use bevy::{prelude::*, sprite::Anchor};
use bevy_procedural_tilemaps::prelude::*;

#[derive(Clone)]
pub struct SpawnableAsset {
    /// Name of the sprite inside our tilemap atlas
    sprite_name: &'static str,
    /// Offset in grid coordinates (for multi-tile objects)
    grid_offset: GridDelta,
    /// Offset in world coordinates (fine positioning)
    offset: Vec3,
    /// Function to add custom components (like collision, physics, etc.)
    components_spawner: fn(&mut EntityCommands),
}

```

**SpawnableAsset Struct**

The `SpawnableAsset` struct contains all the information needed to spawn a tile in our world. The `sprite_name` field gives a name to your sprite (like "grass", "tree", "rock").

The `grid_offset` is used for objects that span multiple tiles - it's a positioning within the tile grid itself. 

For example, the follow tree asset needs four tiles.

<div class="columns is-mobile is-centered">
<div class="column is-narrow">
<table style="border-collapse: separate; border-spacing: 2px; font-family: 'VT323', monospace; font-size: 32px; border-radius: 8px; overflow: hidden; border: none; display: inline-block; margin-bottom: 20px;">
<tr>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important;"><img src="/assets/book_assets/tile_layers/big_tree_1_tl.png" alt="Tree top-left" class="tile-image" style="width: 60px; height: 50px; display: block;"></td>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important;"><img src="/assets/book_assets/tile_layers/big_tree_1_tr.png" alt="Tree top-right" class="tile-image" style="width: 60px; height: 50px; display: block;"></td>
</tr>
<tr>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important;"><img src="/assets/book_assets/tile_layers/big_tree_1_bl.png" alt="Tree bottom-left" class="tile-image" style="width: 60px; height: 50px; display: block;"></td>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important;"><img src="/assets/book_assets/tile_layers/big_tree_1_br.png" alt="Tree bottom-right" class="tile-image" style="width: 60px; height: 50px; display: block;"></td>
</tr>
</table>
</div>
</div>

**Grid Offset**

| Tree Part | Grid Offset | Description |
|-----------|-------------|-------------|
| Bottom-left | `(0, 0)` | Stays at original position |
| Bottom-right | `(1, 0)` | Moves one tile right |
| Top-left | `(0, 1)` | Moves one tile up |
| Top-right | `(1, 1)` | Moves one tile up and right |

<br><br>
The `offset` field, on the other hand, is for fine-tuning the position within the tile - like moving a rock slightly to the left or making sure a tree trunk is perfectly centered within its tile space.

Let's see how `offset` works with rock positioning:

<div class="columns is-mobile is-centered">
<div class="column is-narrow">
<table style="border-collapse: separate; border-spacing: 2px; font-family: 'VT323', monospace; font-size: 32px; border-radius: 8px; overflow: hidden; border: none; display: inline-block; margin-bottom: 20px;">
<tr>
<td style="padding: 3px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important; position: relative;"><img src="/assets/book_assets/tile_layers/rock_1.png" alt="Rock 1" class="tile-image" style="width: 60px; height: 50px; display: block; transform: translate(0px, 0px);"></td>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important; position: relative;"><img src="/assets/book_assets/tile_layers/rock_2.png" alt="Rock 2" class="tile-image" style="width: 60px; height: 50px; display: block; transform: translate(-8px, -6px);"></td>
<td style="padding: 6px; text-align: center; width: 60px; height: 60px; border: none !important; background-color: #c8e6c9; border-radius: 8px !important; position: relative;"><img src="/assets/book_assets/tile_layers/rock_3.png" alt="Rock 3" class="tile-image" style="width: 60px; height: 50px; display: block; transform: translate(6px, 5px);"></td>
</tr>
</table>
</div>
</div>

**Offset**

| Rock | Offset | Description |
|------|--------|-------------|
| Rock 1 | `(0, 0)` | Centered in tile |
| Rock 2 | `(-8, -6)` | Moved slightly left and up |
| Rock 3 | `(6, 5)` | Moved slightly right and down |

Finally, the `components_spawner` is a function that adds custom behavior like collision, physics, or other game mechanics.
<br><br>

**Why is sprite name defined as `&'static str?`**

Let's break down `&'static str` piece by piece to understand why we use it for sprite names.

The `&` symbol means "reference to" - instead of making a new copy of the text, we just note where the original text is located. 

The `'static` is a special lifetime annotation that tells Rust "this text will exist for the entire duration of your game." When you write `"grass"` directly in your code, Rust bakes it into your game file when you build it. It's always there, from game startup to shutdown.


**What's a lifetime and what has `'static` got to do with it?**

A **lifetime** is Rust's way of tracking how long data lives in memory. Rust needs to know when it's safe to use data and when it might be deleted.


Most data has a limited lifetime. For example:
- Local variables live only while a function runs
- Function parameters live only while the function executes
- Data created in a loop might be deleted when the loop ends

But some data lives forever - like string literals embedded in your program. The `'static` lifetime means "this data lives for the entire duration of the program" - it never gets deleted.

This is perfect for our sprite names because they're hardcoded in our source code (like `"grass"`, `"tree"`, `"rock"`) and will never change or be deleted while the program runs. Rust can safely let us use these references anywhere in our code because it knows the data will always be there.

**What's a string literal?**

A string literal is text you write directly in quotes in your code: `"grass"`, `"dirt"`, `"tree"`. 

**Why does Rust need to know when it's safe to use data? Other languages don't seem to care about this.**

Most languages (like C, C++, Java, Python) handle memory safety differently:

- **C/C++**: Don't track lifetimes at all - you can accidentally use deleted data, leading to crashes or security vulnerabilities
- **Java/Python/C#**: Use garbage collection - the runtime automatically deletes unused data, but this adds overhead and unpredictable pauses
- **Rust**: Tracks lifetimes at compile time - prevents crashes without runtime overhead


**The Problem Other Languages Have**
```rust
// Psuedo code warning, don't use
// This would crash in C++ or cause undefined behavior
let sprite_name = {
    let temp = "grass";
    &temp  // temp gets deleted here!
}; 
println!("{}", sprite_name); // CRASH! Using deleted data
```

**Rust Prevents This**
<br>
Rust's compiler analyzes your code and says "Hey, you're trying to use data that might be deleted. I won't let you compile this unsafe code." This catches bugs before your game even runs.

**Does `str` mean String data type?**
<br>
Not quite. `str` represents text data, known as a **string slice**, but you can only use it through a reference like `&str` (a view of text stored somewhere else). `String` is text you own and can modify. Our sprite names like "grass" are baked into the program, so `&str` just points to that text without copying it - much more efficient than using `String`.

`&'static str` means "a reference (`&`), to a string slice (`str`) that lives for the entire program duration (`'static`)." This gives us the best of all worlds: memory efficiency (no copying), performance (direct access), and safety (Rust knows the data will always be valid).


```comic 
left_guy_more_sand: Why does the string need a lifetime? 
right_girl_surprised: Because Rust has trust issues!
```


**What's `GridDelta`?**

`GridDelta` is a struct that represents movement in grid coordinates. It specifies "how many tiles to move" in each direction. For example, `GridDelta::new(1, 0, 0)` means "move one tile to the right", while `GridDelta::new(0, 1, 0)` means "move one tile up". It's used for positioning multi-tile objects like the tree sprite with multiple tiles we mentioned earlier in grid offset.

**Why's components_spawner defined as `fn(&mut EntityCommands)`?**

This is a function pointer that takes a mutable reference to `EntityCommands` (Bevy's way of adding components to entities). Looking at the code in `assets.rs`, we can see it defaults to an empty function that does nothing.

The function pointer allows us to customize what components get added to each spawned entity. For example, a tree sprite might need collision components for physics, while a decorative flower might only need basic rendering components. Each sprite can have its own custom set of components without affecting others.


**Why do we need a mutable reference to EntityCommands?**

Yes! In Rust, you need a mutable reference (`&mut`) when you want to modify something. `EntityCommands` needs to be mutable because it's used to add, remove, or modify components on entities.

Now let's add some helpful methods to our `SpawnableAsset` struct to make it easier to create and configure sprite assets.

Append the following code to the same `assets.rs` file.

```rust
// src/map/assets.rs
impl SpawnableAsset {
    pub fn new(sprite_name: &'static str) -> Self {
        Self {
            sprite_name,
            grid_offset: GridDelta::new(0, 0, 0),
            offset: Vec3::ZERO,
            components_spawner: |_| {}, // Default: no extra components
        }
    }

    pub fn with_grid_offset(mut self, offset: GridDelta) -> Self {
        self.grid_offset = offset;
        self
    }
}
```

**What's `-> Self`?**

In Rust, you must specify the return type of functions (unlike some languages that can infer it). The `-> Self` tells the compiler exactly what type the function returns, which helps catch errors at compile time. `Self` means "the same type as the struct this method belongs to" - so `Self` refers to `SpawnableAsset` here.

**What's `|_| {}`?**

This is a closure (anonymous function) that does nothing. The `|_|` means "takes one parameter but ignores it" (the underscore means we don't use the parameter), and `{}` is an empty function body. 

We need this because our `SpawnableAsset` struct requires a `components_spawner` field (as we saw in the struct definition), but for basic sprites we don't want to add any custom components. This empty closure serves as a "do nothing" default. We'll learn how to use this field to add custom components in later chapters, but for now it's just a placeholder that satisfies the struct's requirements.

**What's a closure? What do you mean by anonymous function?**

A closure is a function that can "capture" variables from its surrounding environment. An anonymous function means it doesn't have a name - you define it inline where you need it, rather than declaring it separately like fn my_function().

**Example usage**

```rust
// Psuedo code warning, don't use
let mut player_health = 100;

// This closure captures 'player_health' by mutable reference
let mut take_damage = || {
    player_health -= 25;  // Modifies the original variable
    player_health
};

// The closure can modify the original variable
take_damage();  // player_health is now 75
take_damage();  // player_health is now 50
```

**Why use closures here?**

In our SpawnableAsset struct closure can be used to allow each sprite to have custom behavior when spawned. For example, a tree might need collision components, while a decorative flower might need different components. The closure can capture game state and configuration to customize spawning behavior for each sprite type.

**Why is semicolon missing in the last line of these functions?**

In Rust, the last expression in a function is automatically returned without needing a `return` keyword or semicolon. This makes it easier to specify what value should be returned - you just write the expression you want to return, and Rust handles the rest. This is Rust's way of making code cleaner and more concise.

**Why can't you manipulate or retrieve `grid_offset` directly?**

The fields of `GridDelta` are private (they have no `pub` keyword), which means they can only be accessed from within the module where `GridDelta` is defined. This is called "encapsulation" - it prevents developers from making mistakes by modifying the struct's data directly, which could break the internal logic. We provide the public method `with_grid_offset()` to safely modify it while maintaining the struct's integrity.

Now that we understand how to define our sprites with `SpawnableAsset`, **how do we load and use these sprites in our game?**

### Loading Sprite Assets

Our game uses a **sprite atlas** - a single large image containing all our sprites. Bevy needs to know where each sprite is located within this image, and we need to avoid reloading the same image multiple times.

Create a folder `tile_layers` in your `src/assets` folder and place `tilemap.png` inside it, you can get it from this [github repo](https://github.com/jamesfebin/ImpatientProgrammerBevyRust). 

<div style="margin: 20px 0; padding: 15px; background-color: #e3f2fd; border-radius: 8px; border-left: 4px solid #1976d2;">
<div style="text-align: center;">
  <img src="/assets/book_assets/tile_layers/tilemap.png" alt="Tilemap sprite atlas" style="max-width: 100%; height: auto; pointer-events: none;">
</div>
The tilemap assets used in this example are based on <a target="_blank" href="https://opengameart.org/content/16x16-game-assets">16x16 Game Assets</a>  by George Bailey, available on OpenGameArt under CC-BY 4.0 license. <strong>However, to follow this tutorial, please use tilemap.png provide from the chapter's <a target="_blank" style="font-weight:650" href="https://github.com/jamesfebin/ImpatientProgrammerBevyRust"> github repo</a>.</strong> 
</div> 

Now inside `src/map` folder create a file `tilemap.rs`. When you add a file inside map folder, ensure to register it in `mod.rs` by adding the line `pub mod tilemap`.

This is where our tilemap definition comes in - it acts as a "map" that tells Bevy the coordinates of every sprite in our atlas.

```rust
// src/map/tilemap.rs
use bevy::math::{URect, UVec2};

pub struct TilemapSprite {
    pub name: &'static str,
    pub pixel_x: u32,
    pub pixel_y: u32,
}

pub struct TilemapDefinition {
    pub tile_width: u32,
    pub tile_height: u32,
    pub atlas_width: u32,
    pub atlas_height: u32,
    pub sprites: &'static [TilemapSprite],
}
```

The `TilemapSprite` struct represents a single sprite within our atlas. It stores the sprite's name (like "dirt" or "green_grass") and its exact pixel coordinates within the atlas image. 

The `TilemapDefinition` struct serves as the "blueprint" that Bevy uses to understand how to slice up our atlas image into individual sprites.

1. **`tile_width` and `tile_height`** - How big each individual sprite is (in our case, 32×32 pixels)
2. **`atlas_width` and `atlas_height`** - The total size of your entire sprite atlas image (the big image containing all sprites)
3. **`sprites`** - The list of all sprites in your atlas, each with its name and location

Though our tilemap stores sprite names and pixel coordinates, Bevy's texture atlas system requires numeric indices and rectangular regions. These methods perform the necessary conversions.


Append the following code to your `tilemap.rs`.

```rust
// src/map/tilemap.rs

impl TilemapDefinition {
    pub const fn tile_size(&self) -> UVec2 {
        UVec2::new(self.tile_width, self.tile_height)
    }

    pub const fn atlas_size(&self) -> UVec2 {
        UVec2::new(self.atlas_width, self.atlas_height)
    }

    pub fn sprite_index(&self, name: &str) -> Option<usize> {
        self.sprites.iter().position(|sprite| sprite.name == name)
    }

    pub fn sprite_rect(&self, index: usize) -> URect {
        let sprite = &self.sprites[index];
        let min = UVec2::new(sprite.pixel_x, sprite.pixel_y);
        URect::from_corners(min, min + self.tile_size())
    }
}
```

The `tile_size()` method converts our tile dimensions into a `UVec2` (unsigned 2D vector), which Bevy uses for size calculations. Similarly, `atlas_size()` provides the total atlas dimensions as a `UVec2`, which Bevy uses to create the texture atlas layout.

The `sprite_index()` method helps in finding sprites by name. When we want to render a "dirt" tile, this method searches through our sprite array and returns the index position of that sprite. 

Finally, `sprite_rect()` takes a sprite index and calculates the exact rectangular region within our atlas that contains that sprite. It uses `URect` (unsigned rectangle) to define the boundaries, which Bevy's texture atlas system requires to know which part of the large image to display.

Now let's put our tilemap definition to use by adding our first sprite - the dirt tile.

### Adding the Dirt Tile

Let's start with a simple dirt tile to test our tilemap system. The dirt tile sits at pixel coordinates (128, 0) in our 256x320 atlas image. We'll add more sprites later as we build out our game world.

Append this code to `tilemap.rs`

```rust
// src/map/tilemap.rs
pub const TILEMAP: TilemapDefinition = TilemapDefinition {
    tile_width: 32,
    tile_height: 32,
    atlas_width: 256,
    atlas_height: 320,
    sprites: &[
          TilemapSprite {
            name: "dirt",
            pixel_x: 128,
            pixel_y: 0,
        },
    ]
};
```

Notice how we're using a const definition - this means all this sprite metadata is determined at compile time. 

### Connecting the Tilemap to Asset Loading

Now that we've defined our tilemap and sprites in `tilemap.rs`, we need to connect this to our asset loading system in `assets.rs`. 

Let's update the imports in `assets.rs` to bring in our `TILEMAP` definition:

```rust
// src/map/assets.rs
use bevy::prelude::*; 
use bevy_procedural_tilemaps::prelude::*;
use crate::map::tilemap::TILEMAP; // <--- line update alert
```

With the import in place, we can now build the three key functions that helps our procedural rendering system:

1. `TilemapHandles` - Container that holds our loaded atlas and layout data
2. `prepare_tilemap_handles` - Loads the atlas image from disk and creates the texture atlas layout defining each sprite's rectangular region
3. `load_assets` - Converts sprite names into `Sprite` data structures ready for rendering

Let's build these step by step.

### Creating the TilemapHandles Struct

First, we need a way to hold references to both the atlas image and its layout. Go ahead and append this code into your `assets.rs`:

```rust
// src/map/assets.rs
#[derive(Clone)]
pub struct TilemapHandles {
    pub image: Handle<Image>,
    pub layout: Handle<TextureAtlasLayout>,
}

impl TilemapHandles {
    pub fn sprite(&self, atlas_index: usize) -> Sprite {
        Sprite::from_atlas_image(
            self.image.clone(),
            TextureAtlas::from(self.layout.clone()).with_index(atlas_index),
        )
    }
}
```

The `TilemapHandles` struct is a container for two handles: `image` points to our loaded sprite sheet file, while `layout` points to the atlas layout that tells Bevy how to slice that image into individual sprites.

The `sprite(atlas_index)` method is a convenience function that creates a ready-to-render `Sprite` by combining the image and layout with a specific index. For example, if the dirt tile is at index 0, calling `tilemap_handles.sprite(0)` gives us a `Sprite` configured to display just the dirt tile from our atlas.

### Loading the Atlas from Disk

Now let's create the function that actually loads the atlas image file and sets up the layout. We will be using our `TILEMAP` definition from earlier.

```rust
// src/map/assets.rs
pub fn prepare_tilemap_handles(
    asset_server: &Res<AssetServer>,
    atlas_layouts: &mut ResMut<Assets<TextureAtlasLayout>>,
    assets_directory: &str,
    tilemap_file: &str,
) -> TilemapHandles {
    let image = asset_server.load::<Image>(format!("{assets_directory}/{tilemap_file}"));
    let mut layout = TextureAtlasLayout::new_empty(TILEMAP.atlas_size());
    for index in 0..TILEMAP.sprites.len() {
        layout.add_texture(TILEMAP.sprite_rect(index));
    }
    let layout = atlas_layouts.add(layout);

    TilemapHandles { image, layout }
}
```

**Breaking it down:**

1. **Load the image**: `asset_server.load()` requests the atlas image file from disk
2. **Create empty layout**: `TextureAtlasLayout::new_empty(TILEMAP.atlas_size())` creates a layout matching our 256x320 atlas
3. **Register each sprite**: The loop iterates through all sprites in `TILEMAP`, using `TILEMAP.sprite_rect(index)` to get each sprite's coordinates and adding them to the layout
4. **Store and return**: The layout is added to Bevy's asset system, and we return a `TilemapHandles` containing both handles

This is where `TILEMAP.atlas_size()` and `TILEMAP.sprite_rect()` from our tilemap definition come into play - they tell Bevy exactly how to slice up our atlas image!

<div style="margin: 20px 0; padding: 15px; background-color: #fff3cd; border-radius: 8px; border-left: 4px solid #ffc107;">
This function loads the atlas into memory and sets up the layout structure, but it doesn't actually generate the game world yet. We're just preparing the tools that the procedural generator will use later to create the map.
</div>

### Converting Sprite Names to Renderable Sprites

Finally, we need a way to convert sprite names (like "dirt") into `Sprite` objects that can be rendered. 

```rust
// src/map/assets.rs
pub fn load_assets(
    tilemap_handles: &TilemapHandles,
    assets_definitions: Vec<Vec<SpawnableAsset>>,
) -> ModelsAssets<Sprite> {
    let mut models_assets = ModelsAssets::<Sprite>::new();
    for (model_index, assets) in assets_definitions.into_iter().enumerate() {
        for asset_def in assets {
            let SpawnableAsset {
                sprite_name,
                grid_offset,
                offset,
                components_spawner,
            } = asset_def;

            let Some(atlas_index) = TILEMAP.sprite_index(sprite_name) else {
                panic!("Unknown atlas sprite '{}'", sprite_name);
            };

            models_assets.add(
                model_index,
                ModelAsset {
                    assets_bundle: tilemap_handles.sprite(atlas_index),
                    grid_offset,
                    world_offset: offset,
                    spawn_commands: components_spawner,
                },
            )
        }
    }
    models_assets
}
```

**Why the two loops?**

Some tiles are simple and need just one sprite (like dirt). Others are complex and need multiple sprites (like a tree that needs 4 parts). 

The outer loop says "for each type of tile," and the inner loop says "for each sprite that tile needs."

**Let's walk through what happens when we load a dirt tile:**

1. We have: `SpawnableAsset { sprite_name: "dirt", ... }`
2. The function asks TILEMAP: "Where is 'dirt'?" → TILEMAP replies: "Index 0"
3. It then asks TilemapHandles: "Give me a Sprite for index 0" → Gets back a `Sprite` object
4. Finally, it packages everything together with the positioning info and stores it

**What does the final data look like?**

After `load_assets` completes, we have a collection of `ModelAsset` objects in memory. Here's what the data structure looks like for a few tiles:

| Model | Field | Value | What It Means |
|-------|-------|-------|---------------|
| **Dirt** | `assets_bundle` | `Sprite(atlas_index: 0)` | Points to dirt sprite in atlas |
|  | `grid_offset` | `(0, 0, 0)` | No grid offset needed |
|  | `world_offset` | `(0, 0, 0)` | No world offset needed |
| **Tree (bottom)** | `assets_bundle` | `Sprite(atlas_index: 31)` | Points to tree bottom sprite |
|  | `grid_offset` | `(0, 0, 0)` | Placed at base position |
|  | `world_offset` | `(0, 0, 0)` | Centered |
| **Tree (top)** | `assets_bundle` | `Sprite(atlas_index: 30)` | Points to tree top sprite |
|  | `grid_offset` | `(0, 1, 0)` | One tile up from bottom |
|  | `world_offset` | `(0, 0, 0)` | Centered |

**Important:** These are just data structures in memory - nothing is drawn on screen yet! The actual rendering happens later when the procedural generator uses these prepared `ModelAsset` objects to spawn entities.


<div style="margin: 20px 0; padding: 15px; background-color: #d4edda; border-radius: 8px; border-left: 4px solid #28a745;">
<strong>Great Progress!</strong> You've made it through the foundation layer - sprites, tilemaps, and asset loading. Now we have the visual pieces (assets), but how does the generator know which tiles can be placed next to each other? That's where models and sockets come in!
</div>

## From Tiles to Models

You already understand **tiles** - the individual visual pieces like grass, dirt, and water. Now we need to build models by adding sockets to these tiles and define connection rules so the generator can figure out valid placements.


### How Models Expose Sockets

Models expose **sockets** - labeled connection points on each edge. Let's look at a green grass model and see how it exposes sockets in different directions.

**Horizontal Plane (x and y directions)**

<div style="display: flex; flex-direction: column; align-items: center; margin: 40px 0; font-family: monospace;">
  <!-- Top socket (y_pos) -->
  <div style="text-align: center; margin-bottom: 15px;">
    <div style="margin-bottom: 10px; font-size: 24px; color: #1976d2;">↑</div>
    <div style="padding: 15px; background-color: #e3f2fd; border-radius: 8px; border-left: 4px solid #1976d2; display: inline-block;">
      <strong>up (y_pos)</strong><br>
      grass.material
    </div>
    <div style="margin-top: 5px; font-size: 24px;">↓</div>
  </div>
  
  <!-- Middle row: left, center, right -->
  <div style="display: flex; align-items: center; gap: 15px; margin: 20px 0;">
    <div style="display: flex; align-items: center; gap: 10px;">
      <div style="font-size: 24px; color: #1976d2;">←</div>
      <div style="padding: 15px; background-color: #e3f2fd; border-radius: 8px; border-left: 4px solid #1976d2;">
        <strong>left (x_neg)</strong><br>
        grass.material
      </div>
    </div>
    
    <div style="font-size: 24px;">→</div>
    
    <div style="padding: 40px 50px; background-color: #90EE90; border-radius: 8px; border-left: 4px solid #28a745; font-size: 18px; font-weight: bold; text-align: center;">
      GREEN<br>GRASS
    </div>
    
    <div style="font-size: 24px;">←</div>
    
    <div style="display: flex; align-items: center; gap: 10px;">
      <div style="padding: 15px; background-color: #e3f2fd; border-radius: 8px; border-left: 4px solid #1976d2;">
        <strong>right (x_pos)</strong><br>
        grass.material
      </div>
      <div style="font-size: 24px; color: #1976d2;">→</div>
    </div>
  </div>
  
  <!-- Bottom socket (y_neg) -->
  <div style="text-align: center; margin-top: 15px;">
    <div style="margin-bottom: 5px; font-size: 24px;">↑</div>
    <div style="padding: 15px; background-color: #e3f2fd; border-radius: 8px; border-left: 4px solid #1976d2; display: inline-block;">
      <strong>down (y_neg)</strong><br>
      grass.material
    </div>
    <div style="margin-top: 10px; font-size: 24px; color: #1976d2;">↓</div>
  </div>
</div>


**Vertical Axis (z direction)**

<div style="display: flex; flex-direction: column; align-items: center; margin: 40px 0; font-family: monospace;">
  <!-- Top socket (z_pos) -->
  <div style="text-align: center; margin-bottom: 15px;">
    <div style="margin-bottom: 10px; font-size: 24px; color: #6c757d;">↑</div>
    <div style="padding: 15px; background-color: #f0f0f0; border-radius: 8px; border-left: 4px solid #6c757d; display: inline-block;">
      <strong>top (z_pos)</strong><br>
      grass.layer_up
    </div>
    <div style="margin-top: 5px; font-size: 24px;">↓</div>
  </div>
  
  <!-- Center tile -->
  <div style="padding: 40px 50px; background-color: #90EE90; border-radius: 8px; border-left: 4px solid #28a745; font-size: 18px; font-weight: bold; text-align: center; margin: 20px 0;">
    GREEN<br>GRASS
  </div>
  
  <!-- Bottom socket (z_neg) -->
  <div style="text-align: center; margin-top: 15px;">
    <div style="margin-bottom: 5px; font-size: 24px;">↑</div>
    <div style="padding: 15px; background-color: #f0f0f0; border-radius: 8px; border-left: 4px solid #6c757d; display: inline-block;">
      <strong>bottom (z_neg)</strong><br>
      grass.layer_down
    </div>
    <div style="margin-top: 10px; font-size: 24px; color: #6c757d;">↓</div>
  </div>
</div>

**How does z-axis make sense in a 2D game?**

Even though we're building a 2D game, the z-axis represents **layering** - Imagine stacking transparent sheets on top of each other. Here's how it works with our yellow grass example:

**The Layering System:**
- **Dirt tiles** form the base layer (ground level)
- **Green grass tiles** can sit on top of dirt (one layer up)
- **Yellow grass tiles** can sit on top of green grass (another layer up)

```comic 
left_guy_sad: I understood sprites... now sockets? 
right_girl_laugh: Think of it as Tinder for tiles!
```

### Building Models

Now that we understand how models expose sockets in all six directions, we need a way to create these models and link them to their visual sprites.

We'll use a helper called `TerrainModelBuilder` that keeps models and sprites paired correctly as we build our world.

### The TerrainModelBuilder 

Create a new file `models.rs` inside the `map` folder, and don't forget to add `pub mod models;` to your `mod.rs`.

```rust
// src/map/models.rs
use bevy_procedural_tilemaps::prelude::*;
use crate::map::assets::SpawnableAsset;

/// Utility wrapper that ensures model declarations and their asset bindings stay aligned.
pub struct TerrainModelBuilder {
    pub models: ModelCollection<Cartesian3D>,
    pub assets: Vec<Vec<SpawnableAsset>>,
}
```

The `TerrainModelBuilder` holds:
1. **`models`**: What the WFC algorithm uses
2. **`assets`**: The sprites for the respective model

Now let's add these methods to the builder.

```rust
// src/map/models.rs
impl TerrainModelBuilder {
    pub fn new() -> Self {
        Self {
            models: ModelCollection::new(),
            assets: Vec::new(),
        }
    }

    pub fn create_model<T>(
        &mut self,
        template: T,
        assets: Vec<SpawnableAsset>,
    ) -> &mut Model<Cartesian3D>
    where
        T: Into<ModelTemplate<Cartesian3D>>,
    {
        let model_ref = self.models.create(template);
        self.assets.push(assets);
        model_ref
    }

    pub fn into_parts(self) -> (Vec<Vec<SpawnableAsset>>, ModelCollection<Cartesian3D>) {
        (self.assets, self.models)
    }
}
```

The `new()` method creates an empty builder to start with. 

The `create_model()` method both a socket definition and the corresponding sprites, then adds them to their respective collections at the same index.

Finally, `into_parts()` splits the builder back into separate collections when you're done building, so the assets can go to the renderer and the models can go to the WFC generator. 

**What's `<T>` doing in `pub fn create_model<T>`?**

The `<T>` is Rust's **generic type parameter** - it's a placeholder that gets filled in with the actual type when you call the function. In our case, we might pass in different types of socket definitions (simple single-socket tiles or complex multi-socket tiles), but we want to perform the same operation on all of them. 

Generics let us write one function that works with multiple types, as long as they can all be converted into a `ModelTemplate`. This is powerful because it means we can add new socket definition types in the future without changing our `TerrainModelBuilder` code.


**What's this `where T: Into<ModelTemplate<Cartesian3D>>`?**

This is a **trait bound** that tells Rust what capabilities the generic type `T` must have. The `where` clause says "T must be able to convert itself into a `ModelTemplate<Cartesian3D>` (a 3D model template)." 

`Into` is Rust's way of saying "this type knows how to transform itself into that type" - like how a string can be converted into a number, or how our socket definitions can be converted into model templates. This means we can pass in any type that knows how to become a `ModelTemplate` - whether it's simple single-socket tiles, complex multi-socket tiles, or even a custom socket type you create later.

This gives us flexibility while ensuring type safety. The compiler will catch any attempts to pass in a type that can't be converted, preventing runtime errors!

## Building the Foundation

Now that we understand how to keep models and assets synchronized, let's start building our procedural world from the ground up. The dirt layer forms the foundation that everything else sits on.

**Layers Make WFC Simpler**

Without layers, we'd need to cram all our rules into a single layer: "water connects to water and grass", "grass connects to grass and dirt", "trees connect to grass", "dirt connects to dirt" - plus all the edge cases and special connections. 

This creates a massive web of interdependencies that makes the WFC algorithm struggle to find valid solutions.

By using layers, we break this complexity into manageable pieces. Each layer only needs to worry about its own connections, making the WFC algorithm much more likely to find valid solutions quickly.

Let's create our dirt layer, make a new file `sockets.rs` inside the `map` folder, and don't forget to add `pub mod sockets;` to your `mod.rs`.

```rust
// src/map/sockets.rs
use bevy_procedural_tilemaps::prelude::*;

pub struct TerrainSockets {
    pub dirt: DirtLayerSockets,
}

pub struct DirtLayerSockets {
    pub layer_up: Socket,      // What can sit on top of dirt
    pub layer_down: Socket,     // What dirt can sit on
    pub material: Socket,       // What dirt connects to horizontally
}
```

The dirt layer needs three types of sockets.

1. **`layer_up`** - This socket handles what can be placed in the layer above dirt. Remember layers are to separate rule cram concerns (water can be above grass without touching it).

2. **`layer_down`** - It handles what layer the dirt itself can be placed on. For the base layer, this will connect to void (empty space).

3. **`material`** - This takes care of horizontal connections between dirt tiles, ensuring they connect properly to form continuous ground.

**Initializing the Sockets**

Now we need to actually create these socket instances. Append this function to `sockets.rs`:

```rust
// src/map/sockets.rs
pub fn create_sockets(socket_collection: &mut SocketCollection) -> TerrainSockets {
    let mut new_socket = || -> Socket { socket_collection.create() };
    
    let sockets = TerrainSockets {
        dirt: DirtLayerSockets {
            layer_up: new_socket(),
            material: new_socket(),
            layer_down: new_socket(),
        },
    };
    sockets
}
```

The `create_sockets` function takes a `SocketCollection` and creates all our socket instances. The `new_socket` closure is a helper that calls `socket_collection.create()` to generate unique socket IDs. Each socket gets a unique identifier that the WFC algorithm uses to track compatibility rules.

### Building the Dirt Layer

Now that we have our socket system defined and initialized, we need to create the rules that tell the WFC algorithm how to use these sockets. This is where we define models and how they connect to each other.

Create a new file `rules.rs` inside the `map` folder, and don't forget to add `pub mod rules;` to your `mod.rs`.

```rust
// src/map/rules.rs
use crate::map::assets::SpawnableAsset;
use crate::map::models::TerrainModelBuilder;
use crate::map::sockets::*;
use bevy_procedural_tilemaps::prelude::*;

fn build_dirt_layer(
    terrain_model_builder: &mut TerrainModelBuilder,
    terrain_sockets: &TerrainSockets,
    socket_collection: &mut SocketCollection,
) {
    terrain_model_builder
        .create_model(
            SocketsCartesian3D::Simple {
                x_pos: terrain_sockets.dirt.material, // right
                x_neg: terrain_sockets.dirt.material, // left
                z_pos: terrain_sockets.dirt.layer_up, // top 
                z_neg: terrain_sockets.dirt.layer_down, // bottom
                y_pos: terrain_sockets.dirt.material, // up
                y_neg: terrain_sockets.dirt.material, // down
            },
            vec![SpawnableAsset::new("dirt")],
        )
        .with_weight(20.);

    socket_collection.add_connections(vec![(
        terrain_sockets.dirt.material,
        vec![terrain_sockets.dirt.material],
    )]);
}
```

**Understanding the Dirt Layer Rules:**

1. **Creates a dirt model** - Defines a tile that **exposes** sockets on all six sides
2. **Exposes socket types** - Horizontal sides expose `dirt.material`, vertical sides expose layer sockets
3. **Assigns a sprite** - `SpawnableAsset::new("dirt")` tells the renderer which sprite to use
4. **Sets the weight** - `.with_weight(20.)` makes dirt tiles 20 times more likely to be placed
5. **Defines connection rules** - `add_connections` tells WFC that `dirt.material` can connect to other `dirt.material`

This creates a simple but effective foundation layer that can form continuous ground while supporting other layers on top!

Now let's append the `build_world` function that the generator will call to get all our dirt layer rules and models:

```rust
// src/map/rules.rs
pub fn build_world() -> (
    Vec<Vec<SpawnableAsset>>,
    ModelCollection<Cartesian3D>,
    SocketCollection,
) {
    let mut socket_collection = SocketCollection::new();
    let terrain_sockets = create_sockets(&mut socket_collection);

    let mut terrain_model_builder = TerrainModelBuilder::new();

    // Build dirt layer
    build_dirt_layer(
        &mut terrain_model_builder,
        &terrain_sockets,
        &mut socket_collection,
    );

    let (assets, models) = terrain_model_builder.into_parts();

    (assets, models, socket_collection)
}
```

**What This Function Does:**

1. **Creates the socket collection** - This is where all our socket connection rules are stored
2. **Gets our socket definitions** - Calls `create_sockets()` to get all the socket types we defined
3. **Creates the model builder** - This keeps our models and assets synchronized
4. **Builds the dirt layer** - Calls our `build_dirt_layer` function to create all the dirt models and rules
5. **Returns the three collections** - Assets for rendering, models for WFC rules, and socket collection for connections

This function is what the generator calls to get all the rules and models needed to create our procedural world!

### Generating Dirt

Now that we have all our components - assets, models, sockets, and rules - we need to configure the procedural generation engine. 

Create a new file `generate.rs` inside the `map` folder, and don't forget to add `pub mod generate;` to your `mod.rs`.

```rust
// src/map/generate.rs
use bevy_procedural_tilemaps::prelude::*;
use bevy::prelude::*;

use crate::map::{
    assets::{load_assets, prepare_tilemap_handles},
    rules::build_world,
};

// -----------------  Configurable values ---------------------------
/// Modify these values to control the map size.
pub const GRID_X: u32 = 25;
pub const GRID_Y: u32 = 18;

// ------------------------------------------------------------------

const ASSETS_PATH: &str = "tile_layers";
const TILEMAP_FILE: &str = "tilemap.png";
/// Size of a block in world units (in Bevy 2d, 1 pixel is 1 world unit)
pub const TILE_SIZE: f32 = 32.;
/// Size of a grid node in world units
const NODE_SIZE: Vec3 = Vec3::new(TILE_SIZE, TILE_SIZE, 1.);

const ASSETS_SCALE: Vec3 = Vec3::ONE;
/// Number of z layers in the map, derived from the default terrain layers.
const GRID_Z: u32 = 1;

pub fn map_pixel_dimensions() -> Vec2 {
    Vec2::new(TILE_SIZE * GRID_X as f32, TILE_SIZE * GRID_Y as f32)
}
```

**Understanding the Configuration Constants:**

Let's break down what each of these constants controls:

1. **`GRID_X` and `GRID_Y`** - These define the size of our generated world in tiles. A 25×18 grid means 450 total tiles (25 × 18 = 450). You can adjust these to create larger or smaller worlds, though larger grids may cause the WFC algorithm to struggle - we'll address scaling issues in a later chapter.

2. **`TILE_SIZE`** - This is the size of each tile in world units. Since we're using 32×32 pixel sprites, each tile takes up 32 world units. This affects how big your world appears on screen.

3. **`NODE_SIZE`** - This tells the generator how much space each grid cell occupies in the 3D world. Equal values = perfect tile fit, smaller NODE_SIZE = overlapping sprites, larger NODE_SIZE = gaps between tiles.

4. **`GRID_Z`** - This defines how many layers our world has. We're currently using 1 layer for dirt, but we'll add more layers later to stack different terrain types on top of each other (dirt, grass, yellow grass, water, props).

5. **`ASSETS_SCALE`** - This controls the size multiplier for sprites. `Vec3::ONE` means sprites render at their original size. 

Now let's append the `setup_generator` function that sets up our procedural generation engine:

```rust
// src/map/generate.rs
pub fn setup_generator(
    mut commands: Commands,
    asset_server: Res<AssetServer>,
    mut atlas_layouts: ResMut<Assets<TextureAtlasLayout>>,
) {
    // 1. Rules Initialization - Get tile definitions and connection rules
    let (assets_definitions, models, socket_collection) = build_world();

    let rules = RulesBuilder::new_cartesian_3d(models, socket_collection)
        // Use ZForward as the up axis (rotation axis for models) since we are using Bevy in 2D
        .with_rotation_axis(Direction::ZForward)
        .build()
        .unwrap();

    // 2. Grid - Create 3D world space with wrapping behavior (false, false, false)
    let grid = CartesianGrid::new_cartesian_3d(GRID_X, GRID_Y, GRID_Z, false, false, false);

    // 3. Configuring the Algorithm - Set up WFC behavior
    let gen_builder = GeneratorBuilder::new()
        .with_rules(rules)
        .with_grid(grid.clone())
        .with_rng(RngMode::RandomSeed)
        .with_node_heuristic(NodeSelectionHeuristic::MinimumRemainingValue)
        .with_model_heuristic(ModelSelectionHeuristic::WeightedProbability);
    
    let generator = gen_builder.build().unwrap();

    // 4. Loading Assets - Load sprite atlas and convert to renderable assets
    let tilemap_handles =
        prepare_tilemap_handles(&asset_server, &mut atlas_layouts, ASSETS_PATH, TILEMAP_FILE);
    let models_assets = load_assets(&tilemap_handles, assets_definitions);

    // 5. Spawning the Generator - Create entity with Transform and NodesSpawner
    commands.spawn((
        Transform::from_translation(Vec3 {
            x: -TILE_SIZE * grid.size_x() as f32 / 2.,
            y: -TILE_SIZE * grid.size_y() as f32 / 2.,
            z: 0.,
        }),
        grid,
        generator,
        NodesSpawner::new(models_assets, NODE_SIZE, ASSETS_SCALE).with_z_offset_from_y(true),
    ));
}
```


**Rules Initialization**

This creates the **constraint solver** that the WFC algorithm uses. It takes our tile definitions and connection rules and converts them into a format the algorithm can understand.

**Why `Direction::ZForward`?**

Since we're building a 2D game, we need to tell the system which axis to use for rotations. `Direction::ZForward` means tiles rotate around the Z-axis (pointing toward/away from the screen), which makes sense for a 2D top-down view.

**Grid**

This creates our world space where tiles will be placed. The three boolean parameters control wrapping behavior:

1. **`(false, false, false)`** - Most games like Minecraft, Terraria (hard boundaries)
2. **`(true, true, false)`** - Classic Asteroids or Pac-Man (wraps left-right and up-down)
3. **`(true, true, true)`** - Advanced simulations with infinite-feeling 3D worlds

**Configuring the Algorithm**

This is where we configure how the WFC algorithm behaves:

1. `RngMode::RandomSeed` - Uses random seeds (same seed = same world every time)
2. `NodeSelectionHeuristic::MinimumRemainingValue` - Always picks the most constrained cell (fewest valid tiles)
3. `ModelSelectionHeuristic::WeightedProbability` - Picks tiles based on their weights (higher weight = more likely)

**Loading Assets and Spawning the Generator**

`prepare_tilemap_handles()` loads our sprite atlas from disk, while `load_assets()` converts our sprite definitions into renderable assets. 

The `commands.spawn()` creates the generator entity with a `Transform` that centers the world on screen and a `NodesSpawner` that handles the actual tile creation.

The `with_z_offset_from_y(true)` setting uses Y coordinates for Z-layer positioning - tiles higher up on screen render in front, creating natural depth ordering (e.g., tree at Y=10 appears in front of rock at Y=5).

#### Final Module Structure

Throughout this chapter, we've been building our procedural generation system across multiple files. Before we integrate everything into your main game, let's make sure your `src/map/mod.rs` file includes all the modules we've created:

```rust
// src/map/mod.rs
pub mod generate;
pub mod assets;
pub mod models;
pub mod rules;
pub mod sockets;
pub mod tilemap;
```

Make sure your `mod.rs` file matches this structure before proceeding to the integration step.

#### Integrating the Generator into Your Game

Now that we've built our procedural generation system, we need to integrate it into our main game. We'll update the `main.rs` file to include the procedural generation plugin and set up the window size to match our generated world.

**Updating main.rs**

We need to add the procedural generation plugin and configure the window size to match our generated world. Update your `main.rs`:

```rust
// src/main.rs
mod map;
mod player;

use bevy::{
    prelude::*,
    window::{Window, WindowPlugin, WindowResolution},
};

use bevy_procedural_tilemaps::prelude::*;

use crate::map::generate::{map_pixel_dimensions, setup_generator};
use crate::player::PlayerPlugin;

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
        .add_systems(Startup, (setup_camera, setup_generator))
        .add_plugins(PlayerPlugin)
        .run();
}

fn setup_camera(mut commands: Commands) {
    commands.spawn(Camera2d);
}
```

**What's New:**

1. **Map module import** - `mod map;` brings in our procedural generation code
2. **Window sizing** - `map_pixel_dimensions()` calculates the window size based on our grid dimensions
3. **Procedural generation plugin** - `ProcGenSimplePlugin` handles the WFC algorithm execution
4. **Generator setup** - `setup_generator` runs at startup to create our world
5. **Image filtering** - `ImagePlugin::default_nearest()` keeps pixel art crisp

**Running Your Procedural World**

Now run your game:

```bash
cargo run
```

You should see a procedurally generated world with dirt tiles following the rules we defined! The world will be centered on screen, and the window size will match your grid dimensions (25×18 tiles = 800×576 pixels).

![Dirt Layer]({{ "/assets/book_assets/chapter2/dirt_layer_without_player.png" | relative_url }})

```comic 
left_guy_smile: IT WORKS! ALL BROWN! BEAUTIFUL! 
right_girl_laugh: Your parents are so proud!
```

**Where's the player?**

The player is actually there, but it's rendering behind the dirt tiles.

We need to make the player render on top of other layers we have.

**Add a Z position constant** 

```rust
// src/player.rs, please it below ANIM_DT const
const PLAYER_Z: f32 = 20.0; 
```

**Update the spawn function** to use this Z value and scale the player slightly down (for better visual proportion with our generated world).

```rust
// src/player.rs - Update the Transform line in spawn_player
Transform::from_translation(Vec3::new(0., 0., PLAYER_Z)).with_scale(Vec3::splat(0.8)),
```

Run your again:

```bash
cargo run
```

Your player renders in front of all tiles and looks proportional to the 32×32 tile world!

![Dirt Layer]({{ "/assets/book_assets/chapter2/dirt_layer_with_player.png" | relative_url }})

## Adding the Grass Layer

Now that we have a working dirt foundation, let's add grass on top. The grass layer will create patches of green grass that sit on the dirt, with proper edge tiles for smooth transitions.

### Step 1: Adding Grass Sprites to the Tilemap

First, we need to add all the grass sprite coordinates to our tilemap. Append these sprites to the `sprites` array in `tilemap.rs`:

```rust
// src/map/tilemap.rs - Add these to the sprites array after the dirt sprite inside TilemapDefinition struct
TilemapSprite {
    name: "green_grass",
    pixel_x: 160,
    pixel_y: 0,
},
TilemapSprite {
    name: "green_grass_corner_in_tl",
    pixel_x: 192,
    pixel_y: 0,
},
TilemapSprite {
    name: "green_grass_corner_in_tr",
    pixel_x: 224,
    pixel_y: 0,
},
TilemapSprite {
    name: "green_grass_corner_in_bl",
    pixel_x: 192,
    pixel_y: 32,
},
TilemapSprite {
    name: "green_grass_corner_in_br",
    pixel_x: 224,
    pixel_y: 32,
},
TilemapSprite {
    name: "green_grass_corner_out_tl",
    pixel_x: 0,
    pixel_y: 64,
},
TilemapSprite {
    name: "green_grass_corner_out_tr",
    pixel_x: 32,
    pixel_y: 64,
},
TilemapSprite {
    name: "green_grass_corner_out_bl",
    pixel_x: 0,
    pixel_y: 96,
},
TilemapSprite {
    name: "green_grass_corner_out_br",
    pixel_x: 32,
    pixel_y: 96,
},
TilemapSprite {
    name: "green_grass_side_t",
    pixel_x: 64,
    pixel_y: 64,
},
TilemapSprite {
    name: "green_grass_side_r",
    pixel_x: 96,
    pixel_y: 64,
},
TilemapSprite {
    name: "green_grass_side_l",
    pixel_x: 64,
    pixel_y: 96,
},
TilemapSprite {
    name: "green_grass_side_b",
    pixel_x: 96,
    pixel_y: 96,
},
```

These sprites include the main grass tile, inner corners, outer corners, and side edges for smooth transitions between grass and dirt.

### Step 2: Adding Grass Sockets

Now we need to define the sockets for the grass layer. Update your `sockets.rs`:

```rust
// src/map/sockets.rs - Add this struct after DirtLayerSockets
pub struct GrassLayerSockets {
    pub layer_up: Socket,
    pub layer_down: Socket,
    pub material: Socket,
    pub void_and_grass: Socket,
    pub grass_and_void: Socket,
    pub grass_fill_up: Socket,
}
```

Then update the `TerrainSockets` struct to include grass:

```rust
// src/map/sockets.rs - Update TerrainSockets
pub struct TerrainSockets {
    pub dirt: DirtLayerSockets,
    pub void: Socket, // line update alert
    pub grass: GrassLayerSockets, // line update alert
}
```

Finally, update the `create_sockets` function to initialize the grass sockets:

```rust
// src/map/sockets.rs - Update create_sockets function
pub fn create_sockets(socket_collection: &mut SocketCollection) -> TerrainSockets {
    let mut new_socket = || -> Socket { socket_collection.create() };
    
    let sockets = TerrainSockets {
        dirt: DirtLayerSockets {
            layer_up: new_socket(),
            material: new_socket(),
            layer_down: new_socket(),
        },
         // line update alert
        void: new_socket(), 
         // lines update alert
        grass: GrassLayerSockets {
            layer_up: new_socket(),
            material: new_socket(),
            layer_down: new_socket(),
            void_and_grass: new_socket(),
            grass_and_void: new_socket(),
            grass_fill_up: new_socket(),
        },
    };
    sockets
}
```

**Why does grass need more sockets than dirt?**

Dirt is simple - it fills the entire base layer, so every dirt tile connects to another dirt tile. Grass is different - it creates patches on top of dirt, which means grass tiles need to handle edges where grass meets empty space.

Here's what each socket handles:

- **`material`** - Connects grass to grass (like dirt's material socket)
- **`layer_up` and `layer_down`** - Vertical connections (like dirt)
- **`void_and_grass`** - Transitions from empty space (left) to grass (right)
- **`grass_and_void`** - Transitions from grass (left) to empty space (right)
- **`grass_fill_up`** - Allows layers above to fill down into grass areas

These transition sockets (`void_and_grass` and `grass_and_void`) are what create smooth edges. Without them, grass patches would have hard, blocky borders instead of the curved corners and sides we want.

### Step 3: Building the Grass Layer Rules

Now let's create the function that builds the grass layer. Append this function to `rules.rs`:

```rust
// src/map/rules.rs - Add this function after build_dirt_layer
fn build_grass_layer(
    terrain_model_builder: &mut TerrainModelBuilder,
    terrain_sockets: &TerrainSockets,
    socket_collection: &mut SocketCollection,
) {
    // Void model - empty space above dirt where no grass exists
    terrain_model_builder.create_model(
        SocketsCartesian3D::Simple {
            x_pos: terrain_sockets.void,
            x_neg: terrain_sockets.void,
            z_pos: terrain_sockets.grass.layer_up,
            z_neg: terrain_sockets.grass.layer_down,
            y_pos: terrain_sockets.void,
            y_neg: terrain_sockets.void,
        },
        Vec::new(),
    );

    // Main grass tile
    terrain_model_builder
        .create_model(
            SocketsCartesian3D::Multiple {
                x_pos: vec![terrain_sockets.grass.material],
                x_neg: vec![terrain_sockets.grass.material],
                z_pos: vec![
                    terrain_sockets.grass.layer_up,
                    terrain_sockets.grass.grass_fill_up,
                ],
                z_neg: vec![terrain_sockets.grass.layer_down],
                y_pos: vec![terrain_sockets.grass.material],
                y_neg: vec![terrain_sockets.grass.material],
            },
            vec![SpawnableAsset::new("green_grass")],
        )
        .with_weight(5.);

    // Outer corner template
    let green_grass_corner_out = SocketsCartesian3D::Simple {
        x_pos: terrain_sockets.grass.void_and_grass,
        x_neg: terrain_sockets.void,
        z_pos: terrain_sockets.grass.layer_up,
        z_neg: terrain_sockets.grass.layer_down,
        y_pos: terrain_sockets.void,
        y_neg: terrain_sockets.grass.grass_and_void,
    }
    .to_template();

    // Inner corner template
    let green_grass_corner_in = SocketsCartesian3D::Simple {
        x_pos: terrain_sockets.grass.grass_and_void,
        x_neg: terrain_sockets.grass.material,
        z_pos: terrain_sockets.grass.layer_up,
        z_neg: terrain_sockets.grass.layer_down,
        y_pos: terrain_sockets.grass.material,
        y_neg: terrain_sockets.grass.void_and_grass,
    }
    .to_template();

    // Side edge template
    let green_grass_side = SocketsCartesian3D::Simple {
        x_pos: terrain_sockets.grass.void_and_grass,
        x_neg: terrain_sockets.grass.grass_and_void,
        z_pos: terrain_sockets.grass.layer_up,
        z_neg: terrain_sockets.grass.layer_down,
        y_pos: terrain_sockets.void,
        y_neg: terrain_sockets.grass.material,
    }
    .to_template();

    // Create rotated versions of outer corners
    terrain_model_builder.create_model(
        green_grass_corner_out.clone(),
        vec![SpawnableAsset::new("green_grass_corner_out_tl")],
    );
    terrain_model_builder.create_model(
        green_grass_corner_out.rotated(ModelRotation::Rot90, Direction::ZForward),
        vec![SpawnableAsset::new("green_grass_corner_out_bl")],
    );
    terrain_model_builder.create_model(
        green_grass_corner_out.rotated(ModelRotation::Rot180, Direction::ZForward),
        vec![SpawnableAsset::new("green_grass_corner_out_br")],
    );
    terrain_model_builder.create_model(
        green_grass_corner_out.rotated(ModelRotation::Rot270, Direction::ZForward),
        vec![SpawnableAsset::new("green_grass_corner_out_tr")],
    );

    // Create rotated versions of inner corners
    terrain_model_builder.create_model(
        green_grass_corner_in.clone(),
        vec![SpawnableAsset::new("green_grass_corner_in_tl")],
    );
    terrain_model_builder.create_model(
        green_grass_corner_in.rotated(ModelRotation::Rot90, Direction::ZForward),
        vec![SpawnableAsset::new("green_grass_corner_in_bl")],
    );
    terrain_model_builder.create_model(
        green_grass_corner_in.rotated(ModelRotation::Rot180, Direction::ZForward),
        vec![SpawnableAsset::new("green_grass_corner_in_br")],
    );
    terrain_model_builder.create_model(
        green_grass_corner_in.rotated(ModelRotation::Rot270, Direction::ZForward),
        vec![SpawnableAsset::new("green_grass_corner_in_tr")],
    );

    // Create rotated versions of side edges
    terrain_model_builder.create_model(
        green_grass_side.clone(),
        vec![SpawnableAsset::new("green_grass_side_t")],
    );
    terrain_model_builder.create_model(
        green_grass_side.rotated(ModelRotation::Rot90, Direction::ZForward),
        vec![SpawnableAsset::new("green_grass_side_l")],
    );
    terrain_model_builder.create_model(
        green_grass_side.rotated(ModelRotation::Rot180, Direction::ZForward),
        vec![SpawnableAsset::new("green_grass_side_b")],
    );
    terrain_model_builder.create_model(
        green_grass_side.rotated(ModelRotation::Rot270, Direction::ZForward),
        vec![SpawnableAsset::new("green_grass_side_r")],
    );

    // Add connection rules
    socket_collection.add_rotated_connection(
        terrain_sockets.dirt.layer_up,
        vec![terrain_sockets.grass.layer_down],
    );
    socket_collection.add_connections(vec![
        (terrain_sockets.void, vec![terrain_sockets.void]),
        (
            terrain_sockets.grass.material,
            vec![terrain_sockets.grass.material],
        ),
        (
            terrain_sockets.grass.void_and_grass,
            vec![terrain_sockets.grass.grass_and_void],
        ),
    ]);
}
```

**Understanding the Grass Layer Function**

This function does several things - let's break it down step by step.

**1. The Void Model - Empty Space**

This creates an "invisible" tile - a spot where no grass grows. Notice `Vec::new()` means no sprite is rendered. The WFC algorithm needs this to create patches of grass rather than covering everything.

**2. The Main Grass Tile**

This is the center grass tile. All four horizontal sides use `grass.material`, meaning they connect to other grass tiles. The `z_pos` has two options - allowing either another layer above OR the special `grass_fill_up` socket for yellow grass later.


A **template** is a reusable socket pattern. Instead of writing the same socket configuration four times (once for each rotation), we create it once and rotate it. The `.to_template()` converts it into a format that can be rotated.

**4. Understanding Rotation - What's Actually Happening?**

We're rotating the **socket pattern**.

Each tile has a sprite (the visual) and sockets (the connection rules). When we rotate a template, the sockets shift to different edges.

<div class="columns is-mobile is-centered">
<div class="column is-narrow">
<div style="text-align: center; margin-bottom: 10px; font-weight: bold; font-size: 16px; font-family: 'Space Mono', monospace;">How Socket Rotation Works</div>
<table style="border-collapse: separate; border-spacing: 4px; font-family: 'Space Mono', monospace; font-size: 13px; border: none; display: inline-block; margin-bottom: 20px;">
<tr>
<td style="padding: 15px; text-align: center; background-color: #e3f2fd; border-radius: 8px;">
<strong style="font-size: 15px;">Original Template (0°)</strong><br><br>
<img src="/assets/book_assets/tile_layers/green_grass_corner_out_tl.png" style="width: 70px; height: 70px; pointer-events: none;"><br><br>
<div style="text-align: left; line-height: 1.6;">
<strong>Sockets:</strong><br>
Left: void<br>
Right: void_and_grass<br>
Forward: void<br>
Backward: grass_and_void
</div>
</td>
<td style="padding: 15px; text-align: center; background-color: #fff3e0; border-radius: 8px;">
<strong style="font-size: 15px;">After 90° Rotation</strong><br><br>
<img src="/assets/book_assets/tile_layers/green_grass_corner_out_bl.png" style="width: 70px; height: 70px; pointer-events: none;"><br><br>
<div style="text-align: left; line-height: 1.6;">
<strong>Sockets:</strong><br>
Left: void<br>
Right: grass_and_void<br>
Forward: void_and_grass<br>
Backward: void
</div>
</td>
</tr>
</table>
</div>
</div>

Notice: the sprites are different (top-left vs bottom-left corner), but the socket pattern shifted clockwise by 90°. The `void_and_grass` socket moved from the right edge to the forward edge.

This is powerful because we define one socket pattern and pair it with different sprites at different rotations. The result: four unique corner models from one template definition.

We do the same for corner inside and side edges of grass tiles as well.


**5. How Simple Rules Create Coherent Shapes**

Here's where the magic happens. We define only three connection rules, yet they create complex, natural-looking grass patches. Let's see how.

**The Three Rules:**

1. **`void` connects to `void`** - Empty space stays empty
2. **`grass.material` connects to `grass.material`** - Grass centers connect to each other
3. **`void_and_grass` connects to `grass_and_void`** - Transition sockets create smooth edges

That's it! But how do these simple rules create coherent grass patches? Let's visualize a 3×3 grass patch forming:

<div class="columns is-mobile is-centered">
<div class="column is-narrow">
<div style="text-align: center; margin-bottom: 12px; font-weight: bold; font-size: 15px; font-family: 'Space Mono', monospace;">How a Grass Patch Forms</div>
<table style="border-collapse: separate; border-spacing: 3px; border: none; display: inline-block; margin-bottom: 20px;">
<tr>
<td style="padding: 0; text-align: center; width: 65px; height: 65px; background-color: #f5f5f5; border-radius: 8px;"></td>
<td style="padding: 0; text-align: center; width: 65px; height: 65px; background-color: #f5f5f5; border-radius: 8px;"></td>
<td style="padding: 0; text-align: center; width: 65px; height: 65px; background-color: #f5f5f5; border-radius: 8px;"></td>
<td style="padding: 0; text-align: center; width: 65px; height: 65px; background-color: #f5f5f5; border-radius: 8px;"></td>
</tr>
<tr>
<td style="padding: 0; text-align: center; width: 65px; height: 65px; background-color: #f5f5f5; border-radius: 8px;"></td>
<td style="padding: 0; text-align: center; width: 65px; height: 65px; background-color: #e8f5e9; border-radius: 8px;"><img src="/assets/book_assets/tile_layers/green_grass_corner_out_tl.png" style="width: 65px; height: 65px; display: block; border-radius: 8px; pointer-events: none;"></td>
<td style="padding: 0; text-align: center; width: 65px; height: 65px; background-color: #e8f5e9; border-radius: 8px;"><img src="/assets/book_assets/tile_layers/green_grass_side_t.png" style="width: 65px; height: 65px; display: block; border-radius: 8px; pointer-events: none;"></td>
<td style="padding: 0; text-align: center; width: 65px; height: 65px; background-color: #e8f5e9; border-radius: 8px;"><img src="/assets/book_assets/tile_layers/green_grass_corner_out_tr.png" style="width: 65px; height: 65px; display: block; border-radius: 8px; pointer-events: none;"></td>
</tr>
<tr>
<td style="padding: 0; text-align: center; width: 65px; height: 65px; background-color: #f5f5f5; border-radius: 8px;"></td>
<td style="padding: 0; text-align: center; width: 65px; height: 65px; background-color: #e8f5e9; border-radius: 8px;"><img src="/assets/book_assets/tile_layers/green_grass_side_l.png" style="width: 65px; height: 65px; display: block; border-radius: 8px; pointer-events: none;"></td>
<td style="padding: 0; text-align: center; width: 65px; height: 65px; background-color: #e8f5e9; border-radius: 8px;"><img src="/assets/book_assets/tile_layers/green_grass.png" style="width: 65px; height: 65px; display: block; border-radius: 8px; pointer-events: none;"></td>
<td style="padding: 0; text-align: center; width: 65px; height: 65px; background-color: #e8f5e9; border-radius: 8px;"><img src="/assets/book_assets/tile_layers/green_grass_side_r.png" style="width: 65px; height: 65px; display: block; border-radius: 8px; pointer-events: none;"></td>
</tr>
<tr>
<td style="padding: 0; text-align: center; width: 65px; height: 65px; background-color: #f5f5f5; border-radius: 8px;"></td>
<td style="padding: 0; text-align: center; width: 65px; height: 65px; background-color: #e8f5e9; border-radius: 8px;"><img src="/assets/book_assets/tile_layers/green_grass_corner_out_bl.png" style="width: 65px; height: 65px; display: block; border-radius: 8px; pointer-events: none;"></td>
<td style="padding: 0; text-align: center; width: 65px; height: 65px; background-color: #e8f5e9; border-radius: 8px;"><img src="/assets/book_assets/tile_layers/green_grass_side_b.png" style="width: 65px; height: 65px; display: block; border-radius: 8px; pointer-events: none;"></td>
<td style="padding: 0; text-align: center; width: 65px; height: 65px; background-color: #e8f5e9; border-radius: 8px;"><img src="/assets/book_assets/tile_layers/green_grass_corner_out_br.png" style="width: 65px; height: 65px; display: block; border-radius: 8px; pointer-events: none;"></td>
</tr>
</table>
</div>
</div>

**Why Every Edge Matches Perfectly:**

Let's trace through the top row to understand how sockets work. Remember: each tile has sockets on its edges that define what can connect to it.

Look at the grass tile on the top left - **green_grass_corner_out_tl** tile (second row, second column in the grid above). This tile has a socket called `void_and_grass` on its **right edge**

Now look at the **green_grass_side_t** tile immediately to its right. This tile has a socket called `grass_and_void` on its **left edge**. When these two tiles sit next to each other, their edges touch. The `void_and_grass` socket (from the corner) connects to the `grass_and_void` socket (from the side) because of Rule 3

The same pattern repeats across the entire grid. **green_grass_side_t** has `grass_and_void` on its **right** edge. **green_grass_corner_out_tr** has `void_and_grass` on its **left** edge. Where they touch, these sockets match perfectly!

The **green_grass** center tile has `material` sockets on all edges, so it connects to any adjacent grass tile that also has `material` on the touching edge.

The WFC algorithm uses these three simple rules to check every tile placement. Before placing a tile, it verifies that all its sockets match the sockets of neighboring tiles. The result: organic-looking grass patches with smooth, curved edges!

**6. Layer Connections**

This tells the WFC algorithm that grass can sit on top of dirt. The `add_rotated_connection` means this rule applies regardless of how the tiles are rotated - grass can always sit on dirt.


### Step 4: Calling the Grass Layer Function

Now update the `build_world` function to call `build_grass_layer`:

```rust
// src/map/rules.rs - Update build_world function
pub fn build_world() -> (
    Vec<Vec<SpawnableAsset>>,
    ModelCollection<Cartesian3D>,
    SocketCollection,
) {
    let mut socket_collection = SocketCollection::new();
    let terrain_sockets = create_sockets(&mut socket_collection);

    let mut terrain_model_builder = TerrainModelBuilder::new();

    // Build dirt layer
    build_dirt_layer(
        &mut terrain_model_builder,
        &terrain_sockets,
        &mut socket_collection,
    );

    // Line update alert
    // Build grass layer
    build_grass_layer(
        &mut terrain_model_builder,
        &terrain_sockets,
        &mut socket_collection,
    );

    let (assets, models) = terrain_model_builder.into_parts();

    (assets, models, socket_collection)
}
```

### Step 5: Updating Grid Layers

Finally, update `generate.rs` to use 2 layers instead of 1:

```rust
// src/map/generate.rs - Update GRID_Z constant
const GRID_Z: u32 = 2;
```

Now run your game:

```bash
cargo run
```

You should see patches of green grass growing on top of the dirt layer, with smooth edges and corners transitioning between grass and dirt!

![Grass Layer]({{ "/assets/book_assets/chapter2/green_grass.png" | relative_url }})

```comic 
left_guy_anxious: How many more layers? 
right_girl_surprised: Enough to make an onion jealous!
```

## Adding the Yellow Grass Layer

Now that we have green grass, let's add yellow grass patches that can grow on top of it! Yellow grass creates visual variety and demonstrates how layers can stack.

### Step 1: Add Yellow Grass Sprites to Tilemap

First, let's add the yellow grass sprites to our tilemap definition. Open `src/map/tilemap.rs` and add these entries to the `sprites` array:

```rust
// src/map/tilemap.rs - Add these to the sprites array after the dirt sprite inside TilemapDefinition struct
TilemapSprite {
    name: "yellow_grass",
    pixel_x: 0,
    pixel_y: 256,
},
TilemapSprite {
    name: "yellow_grass_corner_in_tl",
    pixel_x: 32,
    pixel_y: 256,
},
TilemapSprite {
    name: "yellow_grass_corner_in_tr",
    pixel_x: 64,
    pixel_y: 256,
},
TilemapSprite {
    name: "yellow_grass_corner_in_bl",
    pixel_x: 32,
    pixel_y: 288,
},
TilemapSprite {
    name: "yellow_grass_corner_in_br",
    pixel_x: 64,
    pixel_y: 288,
},
TilemapSprite {
    name: "yellow_grass_corner_out_tl",
    pixel_x: 96,
    pixel_y: 256,
},
TilemapSprite {
    name: "yellow_grass_corner_out_tr",
    pixel_x: 128,
    pixel_y: 256,
},
TilemapSprite {
    name: "yellow_grass_corner_out_bl",
    pixel_x: 96,
    pixel_y: 288,
},
TilemapSprite {
    name: "yellow_grass_corner_out_br",
    pixel_x: 128,
    pixel_y: 288,
},
TilemapSprite {
    name: "yellow_grass_side_t",
    pixel_x: 160,
    pixel_y: 256,
},
TilemapSprite {
    name: "yellow_grass_side_r",
    pixel_x: 192,
    pixel_y: 256,
},
TilemapSprite {
    name: "yellow_grass_side_l",
    pixel_x: 160,
    pixel_y: 288,
},
TilemapSprite {
    name: "yellow_grass_side_b",
    pixel_x: 192,
    pixel_y: 288,
},
```

### Step 2: Define Yellow Grass Sockets

Yellow grass has a special behavior - it sits **on top of green grass**, not on dirt. This means it needs different socket connections.

Add the socket structure to `src/map/sockets.rs`:

```rust
// src/map/sockets.rs - Add after GrassLayerSockets
pub struct YellowGrassLayerSockets {
    pub layer_up: Socket,
    pub layer_down: Socket,
    pub yellow_grass_fill_down: Socket,
}
```

Then update `TerrainSockets` to include yellow grass:

```rust
// src/map/sockets.rs - Update TerrainSockets
pub struct TerrainSockets {
    pub dirt: DirtLayerSockets,
    pub void: Socket,
    pub grass: GrassLayerSockets,
    pub yellow_grass: YellowGrassLayerSockets, // Add this line
}
```

Finally, initialize the yellow grass sockets in `create_sockets`:

```rust
// src/map/sockets.rs - Update create_sockets function
pub fn create_sockets(socket_collection: &mut SocketCollection) -> TerrainSockets {
    let mut new_socket = || -> Socket { socket_collection.create() };
    
    let sockets = TerrainSockets {
        dirt: DirtLayerSockets {
            layer_up: new_socket(),
            material: new_socket(),
            layer_down: new_socket(),
        },
        void: new_socket(),
        grass: GrassLayerSockets {
            layer_up: new_socket(),
            material: new_socket(),
            layer_down: new_socket(),
            void_and_grass: new_socket(),
            grass_and_void: new_socket(),
            grass_fill_up: new_socket(),
        },
        yellow_grass: YellowGrassLayerSockets {
            layer_up: new_socket(),
            layer_down: new_socket(),
            yellow_grass_fill_down: new_socket(),
        },
    };
    sockets
}
```

**Why does yellow grass only need 3 sockets?**

Unlike green grass, yellow grass doesn't need `void_and_grass` transition sockets. Why? Because yellow grass **reuses the green grass edges**. When yellow grass meets empty space, the green grass layer below provides the edge tiles. Yellow grass only appears where green grass already exists, so it uses green grass's `material` socket for horizontal connections.

The `yellow_grass_fill_down` socket is special - it connects to green grass's `grass_fill_up` socket, allowing yellow grass to "fill down" into the green grass layer below.

### Step 3: Building the Yellow Grass Layer Rules

Now let's create the function that builds the yellow grass layer. Add this function to `rules.rs`:

```rust
// src/map/rules.rs - Add this function after build_grass_layer
fn build_yellow_grass_layer(
    terrain_model_builder: &mut TerrainModelBuilder,
    terrain_sockets: &TerrainSockets,
    socket_collection: &mut SocketCollection,
) {
    // Void model - empty space where no yellow grass exists
    terrain_model_builder.create_model(
        SocketsCartesian3D::Simple {
            x_pos: terrain_sockets.void,
            x_neg: terrain_sockets.void,
            z_pos: terrain_sockets.yellow_grass.layer_up,
            z_neg: terrain_sockets.yellow_grass.layer_down,
            y_pos: terrain_sockets.void,
            y_neg: terrain_sockets.void,
        },
        Vec::new(),
    );

    // Main yellow grass tile
    terrain_model_builder
        .create_model(
            SocketsCartesian3D::Simple {
                x_pos: terrain_sockets.grass.material,
                x_neg: terrain_sockets.grass.material,
                z_pos: terrain_sockets.yellow_grass.layer_up,
                z_neg: terrain_sockets.yellow_grass.yellow_grass_fill_down,
                y_pos: terrain_sockets.grass.material,
                y_neg: terrain_sockets.grass.material,
            },
            vec![SpawnableAsset::new("yellow_grass")],
        )
        .with_weight(5.);

    // Outer corner template
    let yellow_grass_corner_out = SocketsCartesian3D::Simple {
        x_pos: terrain_sockets.grass.void_and_grass,
        x_neg: terrain_sockets.void,
        z_pos: terrain_sockets.yellow_grass.layer_up,
        z_neg: terrain_sockets.yellow_grass.yellow_grass_fill_down,
        y_pos: terrain_sockets.void,
        y_neg: terrain_sockets.grass.grass_and_void,
    }
    .to_template();

    // Inner corner template
    let yellow_grass_corner_in = SocketsCartesian3D::Simple {
        x_pos: terrain_sockets.grass.grass_and_void,
        x_neg: terrain_sockets.grass.material,
        z_pos: terrain_sockets.yellow_grass.layer_up,
        z_neg: terrain_sockets.yellow_grass.yellow_grass_fill_down,
        y_pos: terrain_sockets.grass.material,
        y_neg: terrain_sockets.grass.void_and_grass,
    }
    .to_template();

    // Side edge template
    let yellow_grass_side = SocketsCartesian3D::Simple {
        x_pos: terrain_sockets.grass.void_and_grass,
        x_neg: terrain_sockets.grass.grass_and_void,
        z_pos: terrain_sockets.yellow_grass.layer_up,
        z_neg: terrain_sockets.yellow_grass.yellow_grass_fill_down,
        y_pos: terrain_sockets.void,
        y_neg: terrain_sockets.grass.material,
    }
    .to_template();

    // Create rotated versions of outer corners
    terrain_model_builder.create_model(
        yellow_grass_corner_out.clone(),
        vec![SpawnableAsset::new("yellow_grass_corner_out_tl")],
    );
    terrain_model_builder.create_model(
        yellow_grass_corner_out.rotated(ModelRotation::Rot90, Direction::ZForward),
        vec![SpawnableAsset::new("yellow_grass_corner_out_bl")],
    );
    terrain_model_builder.create_model(
        yellow_grass_corner_out.rotated(ModelRotation::Rot180, Direction::ZForward),
        vec![SpawnableAsset::new("yellow_grass_corner_out_br")],
    );
    terrain_model_builder.create_model(
        yellow_grass_corner_out.rotated(ModelRotation::Rot270, Direction::ZForward),
        vec![SpawnableAsset::new("yellow_grass_corner_out_tr")],
    );

    // Create rotated versions of inner corners
    terrain_model_builder.create_model(
        yellow_grass_corner_in.clone(),
        vec![SpawnableAsset::new("yellow_grass_corner_in_tl")],
    );
    terrain_model_builder.create_model(
        yellow_grass_corner_in.rotated(ModelRotation::Rot90, Direction::ZForward),
        vec![SpawnableAsset::new("yellow_grass_corner_in_bl")],
    );
    terrain_model_builder.create_model(
        yellow_grass_corner_in.rotated(ModelRotation::Rot180, Direction::ZForward),
        vec![SpawnableAsset::new("yellow_grass_corner_in_br")],
    );
    terrain_model_builder.create_model(
        yellow_grass_corner_in.rotated(ModelRotation::Rot270, Direction::ZForward),
        vec![SpawnableAsset::new("yellow_grass_corner_in_tr")],
    );

    // Create rotated versions of side edges
    terrain_model_builder.create_model(
        yellow_grass_side.clone(),
        vec![SpawnableAsset::new("yellow_grass_side_t")],
    );
    terrain_model_builder.create_model(
        yellow_grass_side.rotated(ModelRotation::Rot90, Direction::ZForward),
        vec![SpawnableAsset::new("yellow_grass_side_l")],
    );
    terrain_model_builder.create_model(
        yellow_grass_side.rotated(ModelRotation::Rot180, Direction::ZForward),
        vec![SpawnableAsset::new("yellow_grass_side_b")],
    );
    terrain_model_builder.create_model(
        yellow_grass_side.rotated(ModelRotation::Rot270, Direction::ZForward),
        vec![SpawnableAsset::new("yellow_grass_side_r")],
    );

    // Add connection rules
    socket_collection
        .add_rotated_connection(
            terrain_sockets.grass.layer_up,
            vec![terrain_sockets.yellow_grass.layer_down],
        )
        .add_rotated_connection(
            terrain_sockets.yellow_grass.yellow_grass_fill_down,
            vec![terrain_sockets.grass.grass_fill_up],
        );
}
```

Notice how the yellow grass models reuse green grass's transition sockets (`void_and_grass` and `grass_and_void`) for horizontal connections. This is the clever part - yellow grass doesn't define its own edges, it borrows them from green grass!

The connection rules establish two important relationships:
1. `grass.layer_up` connects to `yellow_grass.layer_down` - Yellow grass sits on top of green grass
2. `yellow_grass_fill_down` connects to `grass_fill_up` - This allows yellow grass to appear where green grass has the special "fill up" socket

### Step 4: Calling the Yellow Grass Layer Function

Update `build_world` in `rules.rs` to call `build_yellow_grass_layer`:

```rust
// src/map/rules.rs - Update build_world function
pub fn build_world() -> (
    Vec<Vec<SpawnableAsset>>,
    ModelCollection<Cartesian3D>,
    SocketCollection,
) {
    let mut socket_collection = SocketCollection::new();
    let terrain_sockets = create_sockets(&mut socket_collection);

    let mut terrain_model_builder = TerrainModelBuilder::new();

    // Build dirt layer
    build_dirt_layer(
        &mut terrain_model_builder,
        &terrain_sockets,
        &mut socket_collection,
    );

    // Build grass layer
    build_grass_layer(
        &mut terrain_model_builder,
        &terrain_sockets,
        &mut socket_collection,
    );

    // Line update alert
    // Build yellow grass layer
    build_yellow_grass_layer(
        &mut terrain_model_builder,
        &terrain_sockets,
        &mut socket_collection,
    );

    let (assets, models) = terrain_model_builder.into_parts();

    (assets, models, socket_collection)
}
```

### Step 5: Updating Grid Layers

We need one more layer for yellow grass. Update the constant in `generate.rs`:

```rust
// src/map/generate.rs - Update GRID_Z
const GRID_Z: u32 = 3; // Changed from 2 to 3
```

Now run your game:

```bash
cargo run
```

You should see yellow grass patches appearing on top of green grass, creating a beautiful layered terrain!

![Yellow Grass Layer]({{ "/assets/book_assets/chapter2/yellow_grass.png" | relative_url }})

## Adding the Water Layer

Water adds life to our procedural world! Unlike grass layers that stack on top of each other, water appears **alongside** yellow grass at the same layer level. This creates interesting terrain where water bodies can form near grassy areas.

### Step 1: Add Water Sprites to Tilemap

First, let's add the water sprites to our tilemap definition. Open `src/map/tilemap.rs` and add these entries to the `sprites` array:

```rust
// src/map/tilemap.rs - Add water sprites to the sprites array
TilemapSprite {
    name: "water",
    pixel_x: 32,
    pixel_y: 192,
},
TilemapSprite {
    name: "water_corner_in_tl",
    pixel_x: 64,
    pixel_y: 192,
},
TilemapSprite {
    name: "water_corner_in_tr",
    pixel_x: 96,
    pixel_y: 192,
},
TilemapSprite {
    name: "water_corner_in_bl",
    pixel_x: 64,
    pixel_y: 224,
},
TilemapSprite {
    name: "water_corner_in_br",
    pixel_x: 96,
    pixel_y: 224,
},
TilemapSprite {
    name: "water_corner_out_tl",
    pixel_x: 128,
    pixel_y: 192,
},
TilemapSprite {
    name: "water_corner_out_tr",
    pixel_x: 160,
    pixel_y: 192,
},
TilemapSprite {
    name: "water_corner_out_bl",
    pixel_x: 128,
    pixel_y: 224,
},
TilemapSprite {
    name: "water_corner_out_br",
    pixel_x: 160,
    pixel_y: 224,
},
TilemapSprite {
    name: "water_side_t",
    pixel_x: 192,
    pixel_y: 192,
},
TilemapSprite {
    name: "water_side_r",
    pixel_x: 224,
    pixel_y: 192,
},
TilemapSprite {
    name: "water_side_l",
    pixel_x: 192,
    pixel_y: 224,
},
TilemapSprite {
    name: "water_side_b",
    pixel_x: 224,
    pixel_y: 224,
},
```

### Step 2: Define Water Sockets

Water goes on the next Z-layer above yellow grass. "Layer" here refers to the Z-coordinate in our 3D grid, not geological layers. 

We've been stacking these: dirt at `Z=0`, green grass at `Z=1`, yellow grass at `Z=2`, and now water at `Z=3`. 

At any grid position, you can have dirt at the bottom Z-level and water at a higher Z-level—they occupy the same X,Y position but different Z heights.

Add the socket structure to `src/map/sockets.rs`:

```rust
// src/map/sockets.rs - Add after YellowGrassLayerSockets
pub struct WaterLayerSockets {
    pub layer_up: Socket,
    pub layer_down: Socket,
    pub material: Socket,
    pub void_and_water: Socket,
    pub water_and_void: Socket,
    pub ground_up: Socket,
}
```

Then update `TerrainSockets` to include water:

```rust
// src/map/sockets.rs - Update TerrainSockets
pub struct TerrainSockets {
    pub dirt: DirtLayerSockets,
    pub void: Socket,
    pub grass: GrassLayerSockets,
    pub yellow_grass: YellowGrassLayerSockets,
    pub water: WaterLayerSockets, // Add this line
}
```

Finally, initialize the water sockets in `create_sockets`:

```rust
// src/map/sockets.rs - Update create_sockets function
pub fn create_sockets(socket_collection: &mut SocketCollection) -> TerrainSockets {
    let mut new_socket = || -> Socket { socket_collection.create() };
    
    let sockets = TerrainSockets {
        dirt: DirtLayerSockets {
            layer_up: new_socket(),
            material: new_socket(),
            layer_down: new_socket(),
        },
        void: new_socket(),
        grass: GrassLayerSockets {
            layer_up: new_socket(),
            material: new_socket(),
            layer_down: new_socket(),
            void_and_grass: new_socket(),
            grass_and_void: new_socket(),
            grass_fill_up: new_socket(),
        },
        yellow_grass: YellowGrassLayerSockets {
            layer_up: new_socket(),
            layer_down: new_socket(),
            yellow_grass_fill_down: new_socket(),
        },
        // Line update alert
        water: WaterLayerSockets {
            layer_up: new_socket(),
            layer_down: new_socket(),
            material: new_socket(),
            void_and_water: new_socket(),
            water_and_void: new_socket(),
            ground_up: new_socket(),
        },
    };
    sockets
}
```

Water has 6 sockets because it behaves like green grass - it creates patches with transitions:
1. **`material`** - Connects water to water (like grass's material socket)
2. **`layer_up` and `layer_down`** - Vertical connections
3. **`void_and_water`** and **`water_and_void`** - Transition sockets for smooth edges
4. **`ground_up`** - Special socket that allows props above to know they're not on water

### Step 3: Building the Water Layer Rules

Now let's create the function that builds the water layer. Add this function to `rules.rs`:

```rust
// src/map/rules.rs - Add this function after build_yellow_grass_layer
pub fn build_water_layer(
    terrain_model_builder: &mut TerrainModelBuilder,
    terrain_sockets: &TerrainSockets,
    socket_collection: &mut SocketCollection,
) {
    // Void model - represents land areas where no water exists
    terrain_model_builder.create_model(
        SocketsCartesian3D::Multiple {
            x_pos: vec![terrain_sockets.void],
            x_neg: vec![terrain_sockets.void],
            z_pos: vec![
                terrain_sockets.water.layer_up,
                terrain_sockets.water.ground_up,
            ],
            z_neg: vec![terrain_sockets.water.layer_down],
            y_pos: vec![terrain_sockets.void],
            y_neg: vec![terrain_sockets.void],
        },
        Vec::new(),
    );

    // Main water tile
    const WATER_WEIGHT: f32 = 0.02;
    terrain_model_builder
        .create_model(
            SocketsCartesian3D::Simple {
                x_pos: terrain_sockets.water.material,
                x_neg: terrain_sockets.water.material,
                z_pos: terrain_sockets.water.layer_up,
                z_neg: terrain_sockets.water.layer_down,
                y_pos: terrain_sockets.water.material,
                y_neg: terrain_sockets.water.material,
            },
            vec![SpawnableAsset::new("water")],
        )
        .with_weight(10. * WATER_WEIGHT);

    // Outer corner template
    let water_corner_out = SocketsCartesian3D::Simple {
        x_pos: terrain_sockets.water.void_and_water,
        x_neg: terrain_sockets.void,
        z_pos: terrain_sockets.water.layer_up,
        z_neg: terrain_sockets.water.layer_down,
        y_pos: terrain_sockets.void,
        y_neg: terrain_sockets.water.water_and_void,
    }
    .to_template()
    .with_weight(WATER_WEIGHT);

    // Inner corner template
    let water_corner_in = SocketsCartesian3D::Simple {
        x_pos: terrain_sockets.water.water_and_void,
        x_neg: terrain_sockets.water.material,
        z_pos: terrain_sockets.water.layer_up,
        z_neg: terrain_sockets.water.layer_down,
        y_pos: terrain_sockets.water.material,
        y_neg: terrain_sockets.water.void_and_water,
    }
    .to_template()
    .with_weight(WATER_WEIGHT);

    // Side edge template
    let water_side = SocketsCartesian3D::Simple {
        x_pos: terrain_sockets.water.void_and_water,
        x_neg: terrain_sockets.water.water_and_void,
        z_pos: terrain_sockets.water.layer_up,
        z_neg: terrain_sockets.water.layer_down,
        y_pos: terrain_sockets.void,
        y_neg: terrain_sockets.water.material,
    }
    .to_template()
    .with_weight(WATER_WEIGHT);

    // Create rotated versions of outer corners
    terrain_model_builder.create_model(
        water_corner_out.clone(),
        vec![SpawnableAsset::new("water_corner_out_tl")],
    );
    terrain_model_builder.create_model(
        water_corner_out.rotated(ModelRotation::Rot90, Direction::ZForward),
        vec![SpawnableAsset::new("water_corner_out_bl")],
    );
    terrain_model_builder.create_model(
        water_corner_out.rotated(ModelRotation::Rot180, Direction::ZForward),
        vec![SpawnableAsset::new("water_corner_out_br")],
    );
    terrain_model_builder.create_model(
        water_corner_out.rotated(ModelRotation::Rot270, Direction::ZForward),
        vec![SpawnableAsset::new("water_corner_out_tr")],
    );

    // Create rotated versions of inner corners
    terrain_model_builder.create_model(
        water_corner_in.clone(),
        vec![SpawnableAsset::new("water_corner_in_tl")],
    );
    terrain_model_builder.create_model(
        water_corner_in.rotated(ModelRotation::Rot90, Direction::ZForward),
        vec![SpawnableAsset::new("water_corner_in_bl")],
    );
    terrain_model_builder.create_model(
        water_corner_in.rotated(ModelRotation::Rot180, Direction::ZForward),
        vec![SpawnableAsset::new("water_corner_in_br")],
    );
    terrain_model_builder.create_model(
        water_corner_in.rotated(ModelRotation::Rot270, Direction::ZForward),
        vec![SpawnableAsset::new("water_corner_in_tr")],
    );

    // Create rotated versions of side edges
    terrain_model_builder.create_model(
        water_side.clone(),
        vec![SpawnableAsset::new("water_side_t")],
    );
    terrain_model_builder.create_model(
        water_side.rotated(ModelRotation::Rot90, Direction::ZForward),
        vec![SpawnableAsset::new("water_side_l")],
    );
    terrain_model_builder.create_model(
        water_side.rotated(ModelRotation::Rot180, Direction::ZForward),
        vec![SpawnableAsset::new("water_side_b")],
    );
    terrain_model_builder.create_model(
        water_side.rotated(ModelRotation::Rot270, Direction::ZForward),
        vec![SpawnableAsset::new("water_side_r")],
    );

    // Add connection rules
    socket_collection.add_connections(vec![
        (
            terrain_sockets.water.material,
            vec![terrain_sockets.water.material],
        ),
        (
            terrain_sockets.water.water_and_void,
            vec![terrain_sockets.water.void_and_water],
        ),
    ]);

    // Connect water layer to yellow grass layer
    socket_collection.add_rotated_connection(
        terrain_sockets.yellow_grass.layer_up,
        vec![terrain_sockets.water.layer_down],
    );
}
```

**Key Points About Water:**

1. **Low Weight Values** - Notice `WATER_WEIGHT: f32 = 0.02`. This makes water appear less frequently than grass, creating occasional water bodies instead of covering everything.

2. **Multiple z_pos Options** - The void model has two options for `z_pos`: `water.layer_up` (another water layer could go here) and `water.ground_up` (props can sit here). This prepares us for the props layer we'll add next.

3. **Same Pattern as Grass** - Water uses the same template and rotation approach as grass, demonstrating how the WFC pattern scales to different terrain types.

### Step 4: Calling the Water Layer Function

Update `build_world` in `rules.rs` to call `build_water_layer`:

```rust
// src/map/rules.rs - Update build_world function
pub fn build_world() -> (
    Vec<Vec<SpawnableAsset>>,
    ModelCollection<Cartesian3D>,
    SocketCollection,
) {
    let mut socket_collection = SocketCollection::new();
    let terrain_sockets = create_sockets(&mut socket_collection);

    let mut terrain_model_builder = TerrainModelBuilder::new();

    // Build dirt layer
    build_dirt_layer(
        &mut terrain_model_builder,
        &terrain_sockets,
        &mut socket_collection,
    );

    // Build grass layer
    build_grass_layer(
        &mut terrain_model_builder,
        &terrain_sockets,
        &mut socket_collection,
    );

    // Build yellow grass layer
    build_yellow_grass_layer(
        &mut terrain_model_builder,
        &terrain_sockets,
        &mut socket_collection,
    );

    // Line update alert
    // Build water layer
    build_water_layer(
        &mut terrain_model_builder,
        &terrain_sockets,
        &mut socket_collection,
    );

    let (assets, models) = terrain_model_builder.into_parts();

    (assets, models, socket_collection)
}
```

### Step 5: Updating Grid Layers

We need one more layer for water. Update the constant in `generate.rs`:

```rust
// src/map/generate.rs - Update GRID_Z
const GRID_Z: u32 = 4; // Changed from 3 to 4
```

Now run your game:

```bash
cargo run
```

You should see water bodies forming on your terrain, creating lakes and ponds alongside the grass patches!

![Water Layer]({{ "/assets/book_assets/chapter2/water.png" | relative_url }})

## Adding the Props Layer

Props are the final layer that brings our world to life! Trees, rocks, plants, and stumps should appear on land but not in water. 

This layer sits at the top of our Z-stack and uses special connection rules to ensure props only spawn on solid ground.

### Step 1: Add Props Sprites to Tilemap

First, let's add all the props sprites to our tilemap definition. Open `src/map/tilemap.rs` and add these entries to the `sprites` array:

```rust
// src/map/tilemap.rs - Add these after the water sprites
TilemapSprite {
    name: "big_tree_1_tl",
    pixel_x: 0,
    pixel_y: 0,
},
TilemapSprite {
    name: "big_tree_1_tr",
    pixel_x: 32,
    pixel_y: 0,
},
TilemapSprite {
    name: "big_tree_1_bl",
    pixel_x: 0,
    pixel_y: 32,
},
TilemapSprite {
    name: "big_tree_1_br",
    pixel_x: 32,
    pixel_y: 32,
},
TilemapSprite {
    name: "big_tree_2_tl",
    pixel_x: 64,
    pixel_y: 0,
},
TilemapSprite {
    name: "big_tree_2_tr",
    pixel_x: 96,
    pixel_y: 0,
},
TilemapSprite {
    name: "big_tree_2_bl",
    pixel_x: 64,
    pixel_y: 32,
},
TilemapSprite {
    name: "big_tree_2_br",
    pixel_x: 96,
    pixel_y: 32,
},
TilemapSprite {
    name: "plant_1",
    pixel_x: 128,
    pixel_y: 64,
},
TilemapSprite {
    name: "plant_2",
    pixel_x: 160,
    pixel_y: 64,
},
TilemapSprite {
    name: "plant_3",
    pixel_x: 192,
    pixel_y: 64,
},
TilemapSprite {
    name: "plant_4",
    pixel_x: 224,
    pixel_y: 64,
},
TilemapSprite {
    name: "rock_1",
    pixel_x: 0,
    pixel_y: 128,
},
TilemapSprite {
    name: "rock_2",
    pixel_x: 32,
    pixel_y: 128,
},
TilemapSprite {
    name: "rock_3",
    pixel_x: 64,
    pixel_y: 128,
},
TilemapSprite {
    name: "rock_4",
    pixel_x: 96,
    pixel_y: 128,
},
TilemapSprite {
    name: "small_tree_top",
    pixel_x: 128,
    pixel_y: 128,
},
TilemapSprite {
    name: "small_tree_bottom",
    pixel_x: 128,
    pixel_y: 160,
},
TilemapSprite {
    name: "tree_stump_1",
    pixel_x: 192,
    pixel_y: 128,
},
TilemapSprite {
    name: "tree_stump_2",
    pixel_x: 224,
    pixel_y: 128,
},
TilemapSprite {
    name: "tree_stump_3",
    pixel_x: 0,
    pixel_y: 192,
},
```

### Step 2: Define Props Sockets

Props need special socket handling because they must only appear on land, never in water. They also include multi-tile objects like big trees that span multiple grid positions.

Add the socket structure to `src/map/sockets.rs`:

```rust
// src/map/sockets.rs - Add after WaterLayerSockets
pub struct PropsLayerSockets {
    pub layer_up: Socket,
    pub layer_down: Socket,
    pub props_down: Socket,
    pub big_tree_1_base: Socket,
    pub big_tree_2_base: Socket,
}
```

Then update `TerrainSockets` to include props:

```rust
// src/map/sockets.rs - Update TerrainSockets
pub struct TerrainSockets {
    pub dirt: DirtLayerSockets,
    pub void: Socket,
    pub grass: GrassLayerSockets,
    pub yellow_grass: YellowGrassLayerSockets,
    pub water: WaterLayerSockets,
    pub props: PropsLayerSockets, // Add this line
}
```

Finally, initialize the props sockets in `create_sockets`:

```rust
// src/map/sockets.rs - Update create_sockets function
pub fn create_sockets(socket_collection: &mut SocketCollection) -> TerrainSockets {
    let mut new_socket = || -> Socket { socket_collection.create() };
    
    let sockets = TerrainSockets {
        dirt: DirtLayerSockets {
            layer_up: new_socket(),
            material: new_socket(),
            layer_down: new_socket(),
        },
        void: new_socket(),
        grass: GrassLayerSockets {
            layer_up: new_socket(),
            material: new_socket(),
            layer_down: new_socket(),
            void_and_grass: new_socket(),
            grass_and_void: new_socket(),
            grass_fill_up: new_socket(),
        },
        yellow_grass: YellowGrassLayerSockets {
            layer_up: new_socket(),
            layer_down: new_socket(),
            yellow_grass_fill_down: new_socket(),
        },
        water: WaterLayerSockets {
            layer_up: new_socket(),
            layer_down: new_socket(),
            material: new_socket(),
            void_and_water: new_socket(),
            water_and_void: new_socket(),
            ground_up: new_socket(),
        },
        // Line update alert
        props: PropsLayerSockets {
            layer_up: new_socket(),
            layer_down: new_socket(),
            props_down: new_socket(),
            big_tree_1_base: new_socket(),
            big_tree_2_base: new_socket(),
        },
    };
    sockets
}
```

**Understanding Props Sockets:**

Props have 5 sockets with special purposes:
1. **`layer_up` and `layer_down`** - Standard vertical connections
2. **`props_down`** - Connects to water's `ground_up` socket (ensures props only on land)
3. **`big_tree_1_base` and `big_tree_2_base`** - Special sockets for multi-tile trees that need to connect their base parts

### Step 3: Building the Props Layer Rules

Now let's create the function that builds the props layer. Add this function to `rules.rs`:

```rust
// src/map/rules.rs - Add this function after build_water_layer
pub fn build_props_layer(
    terrain_model_builder: &mut TerrainModelBuilder,
    terrain_sockets: &TerrainSockets,
    socket_collection: &mut SocketCollection,
) {
    // Void model - represents areas where no props exist
    terrain_model_builder.create_model(
        SocketsCartesian3D::Multiple {
            x_pos: vec![terrain_sockets.void],
            x_neg: vec![terrain_sockets.void],
            z_pos: vec![terrain_sockets.props.layer_up],
            z_neg: vec![terrain_sockets.props.layer_down],
            y_pos: vec![terrain_sockets.void],
            y_neg: vec![terrain_sockets.void],
        },
        Vec::new(),
    );

    // Weight constants for different prop types
    const PROPS_WEIGHT: f32 = 0.025;
    const ROCKS_WEIGHT: f32 = 0.008;
    const PLANTS_WEIGHT: f32 = 0.025;
    const STUMPS_WEIGHT: f32 = 0.012;

    // Base prop template - single tile props
    let prop = SocketsCartesian3D::Simple {
        x_pos: terrain_sockets.void,
        x_neg: terrain_sockets.void,
        z_pos: terrain_sockets.props.layer_up,
        z_neg: terrain_sockets.props.props_down,
        y_pos: terrain_sockets.void,
        y_neg: terrain_sockets.void,
    }
    .to_template()
    .with_weight(PROPS_WEIGHT);

    // Create different prop types with different weights
    let plant_prop = prop.clone().with_weight(PLANTS_WEIGHT);
    let stump_prop = prop.clone().with_weight(STUMPS_WEIGHT);
    let rock_prop = prop.clone().with_weight(ROCKS_WEIGHT);

    // Small tree (2 tiles high)
    terrain_model_builder.create_model(
        plant_prop.clone(),
        vec![
            SpawnableAsset::new("small_tree_bottom"),
            SpawnableAsset::new("small_tree_top").with_grid_offset(GridDelta::new(0, 1, 0)),
        ],
    );

    // Big tree 1 (2x2 tiles)
    terrain_model_builder
        .create_model(
            SocketsCartesian3D::Simple {
                x_pos: terrain_sockets.props.big_tree_1_base,
                x_neg: terrain_sockets.void,
                z_pos: terrain_sockets.props.layer_up,
                z_neg: terrain_sockets.props.props_down,
                y_pos: terrain_sockets.void,
                y_neg: terrain_sockets.void,
            },
            vec![
                SpawnableAsset::new("big_tree_1_bl"),
                SpawnableAsset::new("big_tree_1_tl").with_grid_offset(GridDelta::new(0, 1, 0)),
            ],
        )
        .with_weight(PROPS_WEIGHT);

    terrain_model_builder
        .create_model(
            SocketsCartesian3D::Simple {
                x_pos: terrain_sockets.void,
                x_neg: terrain_sockets.props.big_tree_1_base,
                z_pos: terrain_sockets.props.layer_up,
                z_neg: terrain_sockets.props.props_down,
                y_pos: terrain_sockets.void,
                y_neg: terrain_sockets.void,
            },
            vec![
                SpawnableAsset::new("big_tree_1_br"),
                SpawnableAsset::new("big_tree_1_tr").with_grid_offset(GridDelta::new(0, 1, 0)),
            ],
        )
        .with_weight(PROPS_WEIGHT);

    // Big tree 2 (2x2 tiles)
    terrain_model_builder
        .create_model(
            SocketsCartesian3D::Simple {
                x_pos: terrain_sockets.props.big_tree_2_base,
                x_neg: terrain_sockets.void,
                z_pos: terrain_sockets.props.layer_up,
                z_neg: terrain_sockets.props.props_down,
                y_pos: terrain_sockets.void,
                y_neg: terrain_sockets.void,
            },
            vec![
                SpawnableAsset::new("big_tree_2_bl"),
                SpawnableAsset::new("big_tree_2_tl").with_grid_offset(GridDelta::new(0, 1, 0)),
            ],
        )
        .with_weight(PROPS_WEIGHT);

    terrain_model_builder
        .create_model(
            SocketsCartesian3D::Simple {
                x_pos: terrain_sockets.void,
                x_neg: terrain_sockets.props.big_tree_2_base,
                z_pos: terrain_sockets.props.layer_up,
                z_neg: terrain_sockets.props.props_down,
                y_pos: terrain_sockets.void,
                y_neg: terrain_sockets.void,
            },
            vec![
                SpawnableAsset::new("big_tree_2_br"),
                SpawnableAsset::new("big_tree_2_tr").with_grid_offset(GridDelta::new(0, 1, 0)),
            ],
        )
        .with_weight(PROPS_WEIGHT);

    // Tree stumps
    terrain_model_builder.create_model(
        stump_prop.clone(),
        vec![SpawnableAsset::new("tree_stump_1")],
    );
    terrain_model_builder.create_model(
        stump_prop.clone(),
        vec![SpawnableAsset::new("tree_stump_2")],
    );
    terrain_model_builder.create_model(
        stump_prop.clone(),
        vec![SpawnableAsset::new("tree_stump_3")],
    );

    // Rocks
    terrain_model_builder.create_model(rock_prop.clone(), vec![SpawnableAsset::new("rock_1")]);
    terrain_model_builder.create_model(rock_prop.clone(), vec![SpawnableAsset::new("rock_2")]);
    terrain_model_builder.create_model(rock_prop.clone(), vec![SpawnableAsset::new("rock_3")]);
    terrain_model_builder.create_model(rock_prop.clone(), vec![SpawnableAsset::new("rock_4")]);

    // Plants
    terrain_model_builder.create_model(plant_prop.clone(), vec![SpawnableAsset::new("plant_1")]);
    terrain_model_builder.create_model(plant_prop.clone(), vec![SpawnableAsset::new("plant_2")]);
    terrain_model_builder.create_model(plant_prop.clone(), vec![SpawnableAsset::new("plant_3")]);
    terrain_model_builder.create_model(plant_prop.clone(), vec![SpawnableAsset::new("plant_4")]);

    // Add connection rules
    socket_collection.add_connections(vec![
        (
            terrain_sockets.props.big_tree_1_base,
            vec![terrain_sockets.props.big_tree_1_base],
        ),
        (
            terrain_sockets.props.big_tree_2_base,
            vec![terrain_sockets.props.big_tree_2_base],
        ),
    ]);

    // Connect props to water layer
    socket_collection
        .add_rotated_connection(
            terrain_sockets.water.layer_up,
            vec![terrain_sockets.props.layer_down],
        )
        .add_rotated_connection(
            terrain_sockets.props.props_down,
            vec![terrain_sockets.water.ground_up],
        );
}
```

**Key Points About Props:**

1. **Multi-tile Objects** - Big trees use `GridDelta::new(0, 1, 0)` to place the top half one tile up
2. **Weight System** - Different prop types have different spawn probabilities (rocks are rarer than plants)
3. **Land-only Rule** - `props_down` connects to `water.ground_up`, ensuring props never spawn in water
4. **Base Sockets** - Big trees use special base sockets to connect their left and right halves

### Step 4: Calling the Props Layer Function

Update `build_world` in `rules.rs` to call `build_props_layer`:

```rust
// src/map/rules.rs - Update build_world function
pub fn build_world() -> (
    Vec<Vec<SpawnableAsset>>,
    ModelCollection<Cartesian3D>,
    SocketCollection,
) {
    let mut socket_collection = SocketCollection::new();
    let terrain_sockets = create_sockets(&mut socket_collection);

    let mut terrain_model_builder = TerrainModelBuilder::new();

    // Build dirt layer
    build_dirt_layer(
        &mut terrain_model_builder,
        &terrain_sockets,
        &mut socket_collection,
    );

    // Build grass layer
    build_grass_layer(
        &mut terrain_model_builder,
        &terrain_sockets,
        &mut socket_collection,
    );

    // Build yellow grass layer
    build_yellow_grass_layer(
        &mut terrain_model_builder,
        &terrain_sockets,
        &mut socket_collection,
    );

    // Build water layer
    build_water_layer(
        &mut terrain_model_builder,
        &terrain_sockets,
        &mut socket_collection,
    );

    // Line update alert
    // Build props layer
    build_props_layer(
        &mut terrain_model_builder,
        &terrain_sockets,
        &mut socket_collection,
    );

    let (assets, models) = terrain_model_builder.into_parts();

    (assets, models, socket_collection)
}
```

### Step 5: Updating Grid Layers

We need one final layer for props. Update the constant in `generate.rs`:

```rust
// src/map/generate.rs - Update GRID_Z
const GRID_Z: u32 = 5; // Changed from 4 to 5
```

Now run your game:

```bash
cargo run
```

You should see a complete procedural world with dirt, grass, water, and props! Trees and rocks will only appear on land, never in water, creating a realistic and varied landscape.

![Props Layer]({{ "/assets/book_assets/chapter2/props.png" | relative_url }})

### Congratulations!

You've successfully built a complete procedural terrain generation system using Wave Function Collapse! Your world now has:

- **Dirt base layer** - The foundation
- **Green grass patches** - With smooth edges and corners
- **Yellow grass variety** - Stacking on green grass
- **Water bodies** - Creating lakes and ponds
- **Props** - Trees, rocks, and plants that only appear on land

This demonstrates the power of WFC for creating coherent, natural-looking game worlds with just a few simple rules!

## Wait, Just one more thing!

Go to rules.rs and change the water weight.

```rust
const WATER_WEIGHT: f32 = 0.07;
```


Now run your game:

```bash
cargo run
```

![Props Layer]({{ "/assets/book_assets/chapter2/water_weight.png" | relative_url }})

Woah, with a simple modification you are able to change the world to have more water! This demonstrates the power of procedural generation—by tweaking just a few numbers, you can create completely different landscapes. 

Try experimenting with the weight values for different layers to see how dramatically you can transform your world.

Also notice our player can walk on water. We will work on collision detection and also on approaches to build larger maps in our upcoming chapters.

```comic 
left_guy_more_sand: My player walks on water without cheat codes! 
right_girl_laugh: Found Jesus mode!
```

<div style="margin: 20px 0; padding: 15px; background-color: #d4edda; border-radius: 8px; border-left: 4px solid #28a745;">
<strong>Next up</strong> <br> <a href="/posts/bevy-rust-game-development-chapter-3/"> Chapter 3, Let The Data Flow</a>
<br><br>
<strong>Let's stay connected! Here are some ways</strong>
<ul>
<li><a href="https://discord.com/invite/cD9qEsSjUH">Join our community</a> to get notified when new chapters drop. </li>
<li>Follow the project on <a href="https://github.com/jamesfebin/ImpatientProgrammerBevyRust">GitHub</a></li>
<li>Join the discussion on <a href="https://www.reddit.com/r/bevy/comments/1o3y2hr/the_impatient_programmers_guide_to_bevy_and_rust/">Reddit</a></li>
<li>Connect with me on <a href="https://www.linkedin.com/in/febinjohnjames">LinkedIn</a> and <a href="https://x.com/heyfebin">X/Twitter</a></li>
</ul>
</div>

