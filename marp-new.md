Create a new Marp slide deck with a consistent, professional title/content layout. Everything you need — the styling, templates, serving, and PDF export — is embedded in THIS prompt. There are no external theme files or other dependencies: the deck carries its own CSS in an inline `<style>` block.

User's deck topic / request: $ARGUMENTS

## What you're building

A [Marp](https://marp.app/) deck (Markdown → slides) whose defining feature is a **fixed title**: every slide's title (`# H1`) is pinned to the *exact same* top-left position, so it never shifts as body content grows or shrinks. Content is top-aligned below it. This is what makes a deck feel calm and consistent instead of jumpy.

The styling lives in a `<style>` block at the top of the deck's own `.md` file — so the deck is a single self-contained file. Do NOT create a separate theme CSS file.

## Steps

1. **Create the deck** at `slides/<kebab-name>.md`. Start with the exact front-matter + `<style>` block in the "Deck skeleton" section below, then add slides using the layout patterns. Base content on the user's request; if it's thin, draft a sensible outline and build it — don't over-ask.
2. **Serve it** so the user can see it (see "Serving"). Confirm it's up with curl before reporting.
3. **Screenshot** with the browser MCP (navigate to the served URL) and show the user.
4. **Report** the deck path and URL; offer to iterate or export a PDF.

## Deck skeleton (copy this verbatim as the top of the file)

````markdown
---
marp: true
size: 16:9
paginate: true
---

<style>
/* Pin each slide's H1 to the same top-left position on every slide
   so the title never shifts as you advance. */
section > h1 {
  position: absolute;
  top: 60px;
  left: 60px;
  right: 60px;
  margin: 0;
  font-size: 40px;
}

/* Reserve vertical space below the pinned title AND top-align body content.
   Marp's default theme vertically centers block children — override to `start`
   so content hugs the top consistently. */
section {
  padding-top: 140px;
  padding-left: 60px;
  padding-right: 60px;
  align-content: start;
}

/* Cover / title slide: un-pin the H1 and center everything vertically. */
section.cover {
  padding-top: 0;
  align-content: center;
  text-align: center;
}
section.cover > h1 {
  position: static;
  font-size: 96px;
  margin: 0 0 24px 0;
  text-align: center;
}
section.cover .subtitle {
  font-size: 32px;
  color: #666;
  font-weight: 300;
}

/* Dense slide: smaller body text + a 2-column grid for body + side image. */
section.dense { font-size: 22px; }
section.dense .two-col {
  display: grid;
  grid-template-columns: 1fr 280px;
  gap: 32px;
  align-items: start;
}
section.dense .two-col img {
  width: 280px;
  height: 280px;
  object-fit: cover;
  border-radius: 8px;
  border: 1px solid #ccc;
}
section.dense ul { margin: 0; padding-left: 24px; }
section.dense li { margin-bottom: 8px; }
</style>

<!-- _class: cover -->
<!-- _paginate: false -->

# Deck Title Here

<div class="subtitle">A one-line subtitle</div>
````

## Layout patterns (add slides after the skeleton)

**Standard slide** (pinned title + top-aligned body — the workhorse):
```markdown
---

# Slide title stays put

- Body content of any length
- The title above sits in exactly the same spot as every other slide
- Keep to ~5-7 bullets so nothing overflows (Marp does NOT auto-shrink)
```

**Dense slide** (smaller text + two-column body with a side image):
```markdown
---

<!-- _class: dense -->

# A denser slide with an image

<div class="two-col">

- Left column: your bullets
- Right column: the image below
- Great for a diagram, screenshot, or photo alongside points

![alt](https://example.com/image.png)

</div>
```

## Design guidance for kickass slides

- **One idea per slide.** If a slide has two ideas, split it.
- **Titles are claims, not labels.** "Two heads, one repo" beats "Architecture".
- **Bold the payload word** in each bullet so the slide is skimmable at a glance.
- **Respect the overflow rule.** Marp won't shrink overflowing content — keep bodies short while drafting; use `_class: dense` when you genuinely need more.
- **Cover slide uses `_class: cover`; every other slide inherits the pinned title automatically** — no repeated layout markup.
- **Inline SVG or images** for anything structural (arrows, boxes, flows). A diagram beats a bulleted process.

## Serving

Run the Marp server from the workspace root (long-running — let the harness background it):

```bash
npx @marp-team/marp-cli --server --port $PORT slides/
```

- Use `$PORT` from the environment — **do not invent a port number**.
- If another markdown server (md-serve / live-server) already holds `$PORT`, stop it first.
- The Marp server serves a directory index; the deck is at `http://<host>:$PORT/<name>.md`.
- **In the swe-swe container, the browser MCP must use `http://swe-swe:$PORT/...` (NOT localhost).** The user views via their Preview tab.
- Confirm it's up: `curl -s -o /dev/null -w "%{http_code}" http://localhost:$PORT/`

## PDF export (only if the user asks)

No usable system Chrome in this container — point Marp at the bundled Chromium:

```bash
CHROME_PATH=/ms-playwright/chromium-1140/chrome-linux/chrome \
  npx @marp-team/marp-cli --pdf slides/<name>.md -o <name>.pdf \
  --chrome-arg=--no-sandbox
```

If that Chromium path is stale, find it with `ls /ms-playwright/`, or fall back to running `decktape` inside the `*-chrome-1` docker container and `docker cp` the PDF out.
