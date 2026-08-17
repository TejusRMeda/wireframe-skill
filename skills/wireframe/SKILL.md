---
name: wireframe
description: Convert any UI screen into a hand-drawn sketchy wireframe with wobbly lines and handwritten Architects Daughter font. Works with Paper design files, Figma files, or screenshots.
disable-model-invocation: true
---

# Sketchy Wireframe Generator

Transform any polished UI design into a lo-fi hand-drawn wireframe. This skill creates wireframes that feel like napkin sketches — wobbly outlines, handwritten typography, and a warm paper aesthetic.

Works with three kinds of source/target:
- **Paper** design files — read via Paper MCP tools, written back as a new artboard via `write_html`.
- **Figma** design files — read via Figma MCP tools, written back as a new frame via `use_figma` (Plugin API).
- **Screenshots/images** — analyzed visually, written to whichever of Paper or Figma the user is working in (ask if unclear).

## Output Style

- **Background**: Cream paper (#FFFEF9)
- **Strokes**: Dark charcoal (#2D2D2D), 2px weight
- **Font**: Architects Daughter (Google Font) — classic wireframe handwriting
- **Card outlines**: Wobbly SVG/vector paths with slight imperfections, not perfect rectangles
- **Icons**: Simplified sketchy placeholders (boxes with scribbles)
- **Colors**: Preserve semantic colors for status indicators (green=complete, blue=in-progress, red=attention, gray=not-started)

These tokens are identical regardless of target — only the tool calls used to produce them differ (HTML/CSS for Paper, Plugin API JS for Figma).

## Workflow

### Phase 1 — Analyze the Source

1. If given a **Paper** node/artboard:
   - Use `get_screenshot` to capture the design
   - Use `get_tree_summary` to understand the hierarchy
   - Use `get_node_info` and `get_children` to map the structure

2. If given a **Figma** node/frame:
   - Load the `figma-use` skill first — it is a mandatory prerequisite for any `use_figma` call
   - Use `get_screenshot` or `get_design_context` to capture the design and its structure
   - Use `get_metadata` to map the node hierarchy (frames, components, instances, text layers)

3. If given an image/screenshot:
   - Analyze the visual structure
   - Identify: navigation, headers, content areas, cards, buttons, forms, lists, avatars, status badges
   - Ask the user whether the wireframe should be written into Paper or Figma if it isn't already clear from context (e.g. which file is currently open)

4. Document the layout:
   - Overall dimensions and device type (mobile/tablet/desktop)
   - Major sections and their relationships
   - Content hierarchy (headings, body text, captions, labels)
   - Interactive elements (buttons, inputs, links)
   - Data display patterns (cards, tables, lists)

### Phase 2 — Create the Wireframe Container

**Paper:**
```
create_artboard({
  name: "Wireframe - [Original Name]",
  styles: {
    width: "[match source]px",
    height: "[match source]px",
    backgroundColor: "#FFFEF9",
    display: "flex",
    flexDirection: "row" or "column",
    padding: "0px"
  }
})
```

**Figma:**
Use `use_figma` (with `figma-use` in the `skillNames` param) to create a frame on the same page, positioned clear of existing content:
```js
const page = figma.currentPage;
const rightmostX = Math.max(0, ...page.children.map(n => n.x + n.width));
const frame = figma.createFrame();
frame.name = "Wireframe - [Original Name]";
frame.resize([source width], [source height]);
frame.x = rightmostX + 200;
frame.y = 0;
frame.fills = [{ type: 'SOLID', color: { r: 1, g: 0.996, b: 0.976 } }]; // #FFFEF9
return { createdNodeIds: [frame.id] };
```
Use `figma.createAutoLayout(...)` instead of `figma.createFrame()` for any section whose children are structurally related (stacked/side-by-side cards, list rows) rather than positioning everything with absolute x/y.

### Phase 3 — Build Incrementally

Work in small pieces — one visual group at a time. The user sees your work appear live (Paper) or can review after each `use_figma` call (Figma).

#### Wobbly Card/Container Pattern

**Paper (HTML/SVG):**
```html
<svg style="position: absolute; top: 0; left: 0; width: [W]px; height: [H]px;" viewBox="0 0 [W] [H]">
  <path d="M8 4 Q[W/2] 2 [W-8] 5 Q[W-4] [H/2] [W-6] [H-4] Q[W/2] [H-2] 6 [H-5] Q4 [H/2] 8 4"
        fill="#FFFEF9" stroke="#2D2D2D" stroke-width="2" fill-rule="evenodd"/>
</svg>
```

**Figma (Plugin API vector node):** the same wobbly path data can be reused directly — Figma vectors accept SVG path syntax via `vectorPaths`.
```js
const card = figma.createVector();
card.name = "Card Outline";
card.vectorPaths = [{
  windingRule: 'NONZERO',
  data: "M8 4 Q[W/2] 2 [W-8] 5 Q[W-4] [H/2] [W-6] [H-4] Q[W/2] [H-2] 6 [H-5] Q4 [H/2] 8 4"
}];
card.fills = [{ type: 'SOLID', color: { r: 1, g: 0.996, b: 0.976 } }];
card.strokes = [{ type: 'SOLID', color: { r: 0.176, g: 0.176, b: 0.176 } }];
card.strokeWeight = 2;
frame.appendChild(card);
return { createdNodeIds: [card.id] };
```
Vary the control points slightly between cards for organic feel, in both targets.

#### Text with Architects Daughter

**Paper (HTML):**
```html
<span style="font-family: Architects Daughter; font-size: 16px; color: #2D2D2D;">Title Text</span>
<span style="font-family: Architects Daughter; font-size: 13px; color: #666;">Description text</span>
<span style="font-family: Architects Daughter; font-size: 11px; color: #888;">Label or caption</span>
```

**Figma (Plugin API):** follow the canonical text-edit recipe — verify the font is available, load it, `await`, then set characters/size/fills. Never guess the style string.
```js
const available = await figma.listAvailableFontsAsync();
const hasArchitectsDaughter = available.some(f => f.fontName.family === "Architects Daughter");
const fontName = hasArchitectsDaughter
  ? { family: "Architects Daughter", style: "Regular" }
  : { family: "Inter", style: "Regular" }; // fallback if the handwritten font isn't loaded in this file

await figma.loadFontAsync(fontName);
const title = figma.createText();
title.fontName = fontName;
title.characters = "Title Text";
title.fontSize = 16;
title.fills = [{ type: 'SOLID', color: { r: 0.176, g: 0.176, b: 0.176 } }];
frame.appendChild(title);
return { createdNodeIds: [title.id], usedFallbackFont: !hasArchitectsDaughter };
```
If Architects Daughter isn't available in the Figma file, tell the user it fell back to a system font and that they can install the Google Font in Figma for a closer match.

#### Sketchy Icon Placeholder

**Paper (HTML/SVG):**
```html
<svg width="32" height="32" viewBox="0 0 32 32">
  <rect x="2" y="2" width="28" height="28" rx="6" fill="none" stroke="#2D2D2D" stroke-width="1.5" transform="rotate(-1 16 16)"/>
  <!-- Add simple scribble inside -->
  <path d="M10 14 Q16 12 22 16 M8 20 Q16 22 24 18" stroke="#2D2D2D" stroke-width="1" fill="none"/>
</svg>
```

**Figma (Plugin API):**
```js
const icon = figma.createVector();
icon.vectorPaths = [{
  windingRule: 'NONE',
  data: "M10 14 Q16 12 22 16 M8 20 Q16 22 24 18"
}];
icon.strokes = [{ type: 'SOLID', color: { r: 0.176, g: 0.176, b: 0.176 } }];
icon.strokeWeight = 1;
icon.resize(32, 32);
icon.rotation = -1;
frame.appendChild(icon);
```

#### Status Badge Pattern

**Paper (HTML):**
```html
<div style="display: flex; align-items: center; gap: 6px; padding: 3px 10px; border: 1.5px solid #22C55E; border-radius: 10px;">
  <div style="width: 6px; height: 6px; background-color: #22C55E; border-radius: 3px;"></div>
  <span style="font-family: Architects Daughter; font-size: 11px; color: #22C55E;">Complete</span>
</div>
```

**Figma (Plugin API):** an auto-layout row with a small filled ellipse plus loaded-font text.
```js
const badge = figma.createAutoLayout('HORIZONTAL', { itemSpacing: 6, paddingLeft: 10, paddingRight: 10, paddingTop: 3, paddingBottom: 3 });
badge.strokes = [{ type: 'SOLID', color: { r: 0.133, g: 0.773, b: 0.369 } }]; // #22C55E
badge.strokeWeight = 1.5;
badge.cornerRadius = 10;

const dot = figma.createEllipse();
dot.resize(6, 6);
dot.fills = [{ type: 'SOLID', color: { r: 0.133, g: 0.773, b: 0.369 } }];
badge.appendChild(dot);
dot.layoutSizingHorizontal = 'FIXED';
dot.layoutSizingVertical = 'FIXED';

// text: follow the loadFontAsync -> await -> mutate recipe, then appendChild + layoutSizingHorizontal = 'FILL' is NOT needed for label text — leave FIXED/HUG
```

Status colors (same values for both targets):
- Complete/Success: #22C55E (green) → `{r: 0.133, g: 0.773, b: 0.369}`
- In Progress: #3B82F6 (blue) → `{r: 0.231, g: 0.510, b: 0.965}`
- Needs Attention/Error: #EF4444 (red) → `{r: 0.937, g: 0.267, b: 0.267}`
- Not Started/Neutral: #9CA3AF (gray) → `{r: 0.612, g: 0.639, b: 0.686}`

#### Avatar Circle

**Paper (HTML/SVG):**
```html
<svg width="24" height="24" viewBox="0 0 24 24">
  <circle cx="12" cy="12" r="10" fill="#6366F1" stroke="#FFFEF9" stroke-width="2"/>
</svg>
```

**Figma (Plugin API):**
```js
const avatar = figma.createEllipse();
avatar.resize(24, 24);
avatar.fills = [{ type: 'SOLID', color: { r: 0.388, g: 0.4, b: 0.945 } }]; // #6366F1
avatar.strokes = [{ type: 'SOLID', color: { r: 1, g: 0.996, b: 0.976 } }]; // #FFFEF9
avatar.strokeWeight = 2;
frame.appendChild(avatar);
```

#### Button Pattern

**Paper (HTML):**
```html
<!-- Outlined button -->
<div style="padding: 6px 12px; border: 2px solid #2D2D2D; border-radius: 4px;">
  <span style="font-family: Architects Daughter; font-size: 12px; color: #2D2D2D;">Button</span>
</div>

<!-- Filled button -->
<div style="padding: 6px 12px; background-color: #2D2D2D; border-radius: 4px;">
  <span style="font-family: Architects Daughter; font-size: 12px; color: #FFF;">Button</span>
</div>
```

**Figma (Plugin API):** an auto-layout frame around loaded-font text, outlined or filled to match.
```js
const button = figma.createAutoLayout('HORIZONTAL', { paddingLeft: 12, paddingRight: 12, paddingTop: 6, paddingBottom: 6 });
button.cornerRadius = 4;
// outlined:
button.strokes = [{ type: 'SOLID', color: { r: 0.176, g: 0.176, b: 0.176 } }];
button.strokeWeight = 2;
// or filled:
// button.fills = [{ type: 'SOLID', color: { r: 0.176, g: 0.176, b: 0.176 } }];
// then loadFontAsync -> create text child -> appendChild
```

### Phase 4 — Review and Adjust

After building the wireframe, take a screenshot and verify:

1. **Card containment**: All content (avatars, text, buttons) inside their card outlines
2. **Typography legibility**: Text readable at all sizes
3. **Spacing rhythm**: Consistent gaps, nothing cramped
4. **Alignment**: Elements that should align vertically/horizontally do so
5. **Hand-drawn cohesion**: Wobbly lines + Architects Daughter font feel unified

Paper: use `get_screenshot`. Figma: use `await frame.screenshot()` inline, or `get_screenshot` after major milestones. For Figma, also run `get_metadata` to confirm structure (hierarchy, counts, positions) — don't rely on the screenshot alone for structural correctness.

Fix any issues before finishing. For Figma, keep edits incremental (≤10 logical operations per `use_figma` call) and always return created/mutated node IDs so follow-up calls can target them.

### Phase 5 — Finish

**Paper:** call `finish_working_on_nodes()` when complete.
**Figma:** no equivalent close-out call — just confirm the final `screenshot()`/`get_screenshot` looks right and report the frame name/ID to the user.

## Size Guidelines

Adjust card/container heights to fit content:
- Small card (icon + title + description): ~180-200px
- Medium card (with avatars/metadata): ~220px
- Large card (with lists/details): ~280px+

Always ensure SVG/vector backgrounds are tall enough to contain all card content.

## Example Invocations

| Input | Action |
|-------|--------|
| "wireframe this screen" + Paper selection | Convert selected artboard/node to wireframe, written back into Paper |
| "wireframe this screen" + Figma selection | Convert selected frame/node to wireframe, written back into Figma |
| "wireframe" + image URL or path | Analyze image and create wireframe in whichever tool (Paper/Figma) is currently open |
| "make a sketchy version of this" | Same as above |
| "convert to hand-drawn wireframe" | Same as above |

## Tone

You are a designer quickly sketching ideas on paper. The wireframe should feel:
- **Quick**: Like it was drawn in a few minutes
- **Approachable**: Not intimidating or overly polished
- **Clear**: Structure and hierarchy are obvious despite the loose style
- **Warm**: The cream paper and handwriting feel human and inviting
