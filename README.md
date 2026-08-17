# wireframe — sketchy wireframe generator (Claude Skill)

Turn any polished UI screen into a hand-drawn, napkin-sketch wireframe — wobbly SVG outlines, handwritten typography, warm cream paper. Works straight from a [Paper](https://app.paper.design) design file or from a plain screenshot.

## What it does

Feed it a Paper artboard/node or an image, and it produces a new artboard that looks like a designer sketched the same screen by hand:

- Cream paper background (`#FFFEF9`), dark charcoal wobbly strokes
- Handwritten `Architects Daughter` type for all text
- Sketchy icon, button, avatar, and status-badge placeholders
- Semantic status colors preserved (green = complete, blue = in progress, red = attention, gray = not started)

It's a genuinely useful step in a design review or a pitch deck: it strips the polish so people react to structure and flow instead of pixel-level detail.

## Install

```bash
npx skills add TejusRMeda/wireframe-skill
```

Or grab just the skill file directly:

```bash
npx skills add https://github.com/TejusRMeda/wireframe-skill/tree/main/skills/wireframe
```

This drops `SKILL.md` into your agent's skills directory (e.g. `~/.claude/skills/wireframe` for Claude Code).

## Use it

The skill only runs when explicitly invoked (it won't auto-trigger on its description), so ask for it directly:

- "wireframe this screen" (with a Paper selection)
- "wireframe" + an image path or URL
- "make a sketchy version of this"
- "convert to hand-drawn wireframe"

It needs the [Paper MCP server](https://paper.design) connected to read Paper files; screenshots/images work with no extra setup.

## How it works

1. **Analyze the source** — reads the Paper node tree (or the image) to map navigation, cards, forms, and content hierarchy.
2. **Create a matching artboard** — same dimensions as the source, cream background.
3. **Build incrementally** — writes HTML/SVG in small visual groups: wobbly card outlines, handwritten text, sketchy icons, status badges, buttons, avatars.
4. **Review** — screenshots the result and checks containment, spacing, alignment, and legibility.
5. **Finish** — hands the artboard back.

Full prompt logic lives in [`skills/wireframe/SKILL.md`](skills/wireframe/SKILL.md).

## License

MIT
