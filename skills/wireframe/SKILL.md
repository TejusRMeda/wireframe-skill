---
name: wireframe
description: Convert any UI screen into a hand-drawn sketchy wireframe with wobbly lines and handwritten Architects Daughter font. Works with Paper design files or screenshots.
disable-model-invocation: true
---

# Sketchy Wireframe Generator

Transform any polished UI design into a lo-fi hand-drawn wireframe. This skill creates wireframes that feel like napkin sketches — wobbly outlines, handwritten typography, and a warm paper aesthetic.

## Output Style

- **Background**: Cream paper (#FFFEF9)
- **Strokes**: Dark charcoal (#2D2D2D), 2px weight
- **Font**: Architects Daughter (Google Font) — classic wireframe handwriting
- **Card outlines**: Wobbly SVG paths with slight imperfections, not perfect rectangles
- **Icons**: Simplified sketchy placeholders (boxes with scribbles)
- **Colors**: Preserve semantic colors for status indicators (green=complete, blue=in-progress, red=attention, gray=not-started)

## Workflow

### Phase 1 — Analyze the Source

1. If given a Paper node/artboard:
   - Use `get_screenshot` to capture the design
   - Use `get_tree_summary` to understand the hierarchy
   - Use `get_node_info` and `get_children` to map the structure

2. If given an image/screenshot:
   - Analyze the visual structure
   - Identify: navigation, headers, content areas, cards, buttons, forms, lists, avatars, status badges

3. Document the layout:
   - Overall dimensions and device type (mobile/tablet/desktop)
   - Major sections and their relationships
   - Content hierarchy (headings, body text, captions, labels)
   - Interactive elements (buttons, inputs, links)
   - Data display patterns (cards, tables, lists)

### Phase 2 — Create the Wireframe Artboard

1. Create a new artboard with matching dimensions:
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

### Phase 3 — Build Incrementally

Write HTML in small pieces — one visual group at a time. The user sees your work appear live.

#### Wobbly Card/Container Pattern
Use SVG paths with quadratic curves for hand-drawn rectangles:
```html
<svg style="position: absolute; top: 0; left: 0; width: [W]px; height: [H]px;" viewBox="0 0 [W] [H]">
  <path d="M8 4 Q[W/2] 2 [W-8] 5 Q[W-4] [H/2] [W-6] [H-4] Q[W/2] [H-2] 6 [H-5] Q4 [H/2] 8 4"
        fill="#FFFEF9" stroke="#2D2D2D" stroke-width="2" fill-rule="evenodd"/>
</svg>
```
Vary the control points slightly between cards for organic feel.

#### Text with Architects Daughter
```html
<span style="font-family: Architects Daughter; font-size: 16px; color: #2D2D2D;">Title Text</span>
<span style="font-family: Architects Daughter; font-size: 13px; color: #666;">Description text</span>
<span style="font-family: Architects Daughter; font-size: 11px; color: #888;">Label or caption</span>
```

#### Sketchy Icon Placeholder
```html
<svg width="32" height="32" viewBox="0 0 32 32">
  <rect x="2" y="2" width="28" height="28" rx="6" fill="none" stroke="#2D2D2D" stroke-width="1.5" transform="rotate(-1 16 16)"/>
  <!-- Add simple scribble inside -->
  <path d="M10 14 Q16 12 22 16 M8 20 Q16 22 24 18" stroke="#2D2D2D" stroke-width="1" fill="none"/>
</svg>
```

#### Status Badge Pattern
```html
<div style="display: flex; align-items: center; gap: 6px; padding: 3px 10px; border: 1.5px solid #22C55E; border-radius: 10px;">
  <div style="width: 6px; height: 6px; background-color: #22C55E; border-radius: 3px;"></div>
  <span style="font-family: Architects Daughter; font-size: 11px; color: #22C55E;">Complete</span>
</div>
```

Status colors:
- Complete/Success: #22C55E (green)
- In Progress: #3B82F6 (blue)
- Needs Attention/Error: #EF4444 (red)
- Not Started/Neutral: #9CA3AF (gray)

#### Avatar Circle
```html
<svg width="24" height="24" viewBox="0 0 24 24">
  <circle cx="12" cy="12" r="10" fill="#6366F1" stroke="#FFFEF9" stroke-width="2"/>
</svg>
```

#### Button Pattern
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

### Phase 4 — Review and Adjust

After building the wireframe, take a screenshot and verify:

1. **Card containment**: All content (avatars, text, buttons) inside their card outlines
2. **Typography legibility**: Text readable at all sizes
3. **Spacing rhythm**: Consistent gaps, nothing cramped
4. **Alignment**: Elements that should align vertically/horizontally do so
5. **Hand-drawn cohesion**: Wobbly lines + Architects Daughter font feel unified

Fix any issues before finishing.

### Phase 5 — Finish

Call `finish_working_on_nodes()` when complete.

## Size Guidelines

Adjust card/container heights to fit content:
- Small card (icon + title + description): ~180-200px
- Medium card (with avatars/metadata): ~220px
- Large card (with lists/details): ~280px+

Always ensure SVG backgrounds are tall enough to contain all card content.

## Example Invocations

| Input | Action |
|-------|--------|
| "wireframe this screen" + Paper selection | Convert selected artboard/node to wireframe |
| "wireframe" + image URL or path | Analyze image and create wireframe |
| "make a sketchy version of this" | Same as above |
| "convert to hand-drawn wireframe" | Same as above |

## Tone

You are a designer quickly sketching ideas on paper. The wireframe should feel:
- **Quick**: Like it was drawn in a few minutes
- **Approachable**: Not intimidating or overly polished
- **Clear**: Structure and hierarchy are obvious despite the loose style
- **Warm**: The cream paper and handwriting feel human and inviting
