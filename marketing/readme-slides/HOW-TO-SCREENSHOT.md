# README Slides — How to Screenshot

Five standalone visual assets for the project README.

## View

Open `index.html` in any browser. Or:

```bash
open marketing/readme-slides/index.html
```

## Navigation

| Key            | Action                              |
| -------------- | ----------------------------------- |
| `↓` / `→` / `Space` / `j` | Next slide                |
| `↑` / `←` / `k`           | Previous slide            |
| `Home` / `End`            | First / last slide        |
| `H`                       | Toggle UI chrome (nav dots + hint) — hide before screenshot |

## Recommended screenshot setup

1. **Window size**: 1440 × 900 or larger. For retina-sharp output, screenshot at 2560 × 1440.
2. **Browser**: Chrome or Safari with no extensions visible. Toggle into fullscreen (`Cmd+Ctrl+F` on macOS) for cleanest framing.
3. **Hide UI**: press `H` to hide the right-side nav dots and bottom keyboard hint before each shot.
4. **Capture**: macOS `Cmd+Shift+4` then `Space` then click the page area. Or use `Cmd+Shift+5` and select the window.
5. **Save** screenshots into `marketing/screenshots/` with names matching the README placeholders:

```
marketing/screenshots/01-hero.png
marketing/screenshots/02-pain.png
marketing/screenshots/03-status.png
marketing/screenshots/04-flow.png
marketing/screenshots/05-phases.png
```

## Then update the README

The `README.md` already has placeholders:

```html
<!-- SCREENSHOT: 01-hero.png -->
```

Replace each with the markdown image embed:

```markdown
![](./marketing/screenshots/01-hero.png)
```

## Style notes

- Palette: warm cream paper (`#F5F2EC`), sumi black (`#1A1A1A`), vermilion seal (`#A8332A`).
- Typography: Fraunces (display serif, editorial), IBM Plex Sans (body), JetBrains Mono (code).
- Aesthetic: Japanese magazine minimalism — hairlines, asymmetric grid, generous whitespace, no shadows or gradients.

If anything feels off, edit `index.html` directly; everything is inline (zero dependencies, no build step).
