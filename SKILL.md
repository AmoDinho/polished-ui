---
name: polished-ui
description: Build or upgrade web UI that looks custom, not AI-generated. Hallmark sets the design direction. UI SDK (React 19+ / Next.js) supplies accessible controls and custom behaviors. Kokonut UI, Magic UI, beUI and Obsidian UI supply visual parts only. Use for any new page, landing page, app screen, redesign, UI polish, or a request to set up ui-sdk or build a control, form, dialog, menu, or custom widget.
---

# Polished UI

One workflow that wraps six sources:

| Source | What it gives you | How you reach it |
|---|---|---|
| **Hallmark** (nutlope/hallmark) | Design direction: 50 themes or a custom direction, layout built from the page's job, anti-slop gates, `sloplint` browser audit, fresh-eyes review | Separate skill: `/hallmark`, `scripts/sloplint.mjs` |
| **UI SDK** (ui-sdk.dev, vercel-labs/ui) | Accessible interaction: keyboard, focus, popups, lists, forms, and a way to build a control the library doesn't ship. Parts are unstyled; `web.css` is only a baseline the page's CSS overrides | `@ui-sdk/react` + `@ui-sdk/core`. Skill: `npx skills add vercel-labs/ui --skill ui-sdk` |
| **Kokonut UI** (kokonutui.com) | 100+ product components: AI inputs, loaders, cards, drawers, text effects, backgrounds | shadcn registry `@kokonutui/<name>` |
| **Magic UI** (magicui.design) | Marketing-page effects: marquee, bento grid, globe, beams, borders, number tickers, device mockups | shadcn registry `@magicui/<name>` |
| **beUI** (beui.dev) | Interaction-grade motion primitives, blocks, AI-agent UI, charts | shadcn registry `@beui/<name>`, MCP at `https://mcp.beui.dev/mcp` |
| **Obsidian UI** (obsidianui.dev) | A small, editorial set of GSAP-driven showpieces: image hover galleries, parallax and split showcases, draggable marquee, text reels, plus dashboard blocks | shadcn registry `@obsidian/<name>` or direct URL |

The rule that holds it together: **Hallmark decides what the page is and how it looks. The libraries only supply parts.** UI SDK may own how a control behaves (focus, keys, ARIA). It does not own colour, type, or layout. Never let a component library drive the page, and never add an effect because it exists.

---

## 0. Pre-flight (always)

1. **Read the project.** Framework (Next.js / Vite / Remix / plain HTML), React version, Tailwind version, whether `components.json` exists, existing fonts, palette, tokens, and any `design.md` / `DESIGN.md`. A locked design file wins over everything below.
2. **Pick the lane:**
   - **Empty folder, and the user wants an app or asked to set up ui-sdk** → create the Next.js app first (UI SDK setup below), then install UI SDK. Don't invent a page if they only asked for setup.
   - **React 19+ project** (Next.js, Vite, Remix) → full workflow. UI SDK for controls people use; the styled registries only for visual parts.
   - **React 18 or older** → UI SDK will not install cleanly. Say so before adding packages. Use beUI / the project's existing controls until they upgrade. Don't convert the app to Next.js to get around it.
   - **Plain HTML / non-React** → Hallmark on its own. The other sources are React (UI SDK also has Vue, Svelte, Solid, and web components, but this skill does not set those up). You may port one effect to vanilla CSS/JS by reading its source. Don't add React unless the user asked for an app.
   - **React Native / Expo** → beUI Native (`npx beui-native add <name>`) after its one-time setup; skip UI SDK, Magic UI, Kokonut and Obsidian UI.
3. **Ask once for real material**: copy, product screenshots, photos, real numbers, logos. Never invent testimonials, metrics, customer logos, or prices. Use clearly marked placeholders if the user has nothing yet.
4. **Mode.** Is this a new build, a redesign of something that exists, or an audit only? Never restyle an existing page unless the user named it.

## 1. Make sure the tools are installed

Check, don't assume. Install only what's missing.

**Hallmark** (skill + linter):
```bash
ls ~/.claude/skills/hallmark 2>/dev/null || ls .claude/skills/hallmark 2>/dev/null \
  || npx skills add nutlope/hallmark -g
# Browser checks need puppeteer-core + Chrome; static checks run without them
npm ls -g puppeteer-core >/dev/null 2>&1 || npm i -g puppeteer-core
```
Needs Node 18+. If the user doesn't want a global install, drop `-g`.

**shadcn base** (React projects without `components.json`):
```bash
npx shadcn@latest init
```
Kokonut UI and beUI target **React 19 + Tailwind CSS v4 + Motion**. If the project is on Tailwind v3 or React 18, tell the user before installing; some components will need adjusting. Don't run `shadcn init` just to use UI SDK. It is not a registry.

**Registries** — merge into `components.json`, keeping any entries already there:
```json
{
  "registries": {
    "@kokonutui": "https://kokonutui.com/r/{name}.json",
    "@magicui": "https://magicui.design/r/{name}.json",
    "@beui": "https://beui.dev/r/{name}.json",
    "@obsidian": "https://www.obsidianui.dev/r/{name}.json"
  }
}
```
Add `@obsidian` only when you plan to use an Obsidian UI component. If the namespace doesn't resolve, install by direct URL instead, which is what its docs show: `npx shadcn@latest add "https://www.obsidianui.dev/r/<name>.json"`.

Kokonut also needs its `cn` helper once:
```bash
npx shadcn@latest add https://kokonutui.com/r/utils.json
```

**Optional MCP servers** (faster discovery inside the editor; suggest, don't force):
```bash
npx shadcn@latest mcp init --client claude      # searches all registries in components.json
claude mcp add --transport http beui https://mcp.beui.dev/mcp
```

Match the project's package manager (`bunx --bun shadcn@latest`, `pnpm dlx shadcn@latest`, `npx shadcn@latest`). In monorepos pass `-c ./apps/<app>`.

**UI SDK** — set this up when the page has controls people use (forms, menus, dialogs, tabs, selects, anything custom), or when the user asked to set up ui-sdk. Skip it on a static marketing page that only needs a marquee. The styled registries fight a custom direction; UI SDK does not, because its parts take the page's CSS. Setup exists so the next step reads the installed docs instead of inventing ARIA.

Do these in order. Check first; don't reinstall what's already there.

1. **Empty folder.** Create the app in the current directory, then continue inside it:
   ```bash
   npx create-next-app@latest .
   ```
   Accept App Router, TypeScript, Tailwind, and React 19. If the CLI would wait for prompts, pass its non-interactive yes flag and the same choices. Don't scaffold a nested folder, and don't run this in a repo that already has an app.
2. **React 19.** `@ui-sdk/react` needs React and React DOM 19 or newer. If either is older, stop and tell the user — don't install anyway. A TypeScript project also needs `@types/react` 19.2.10 or newer.
3. **Packages.** Install `@ui-sdk/react` and `@ui-sdk/core` with the project's package manager (`pnpm add`, `yarn add`, `bun add`, or `npm install`). Core is a peer of the React package; both must resolve to one copy.
4. **Baseline CSS, once, before the app's own CSS.** In the app entry (Next.js App Router: the root layout, above the globals import):
   ```ts
   import "@ui-sdk/core/web.css"
   ```
   `web.css` is a default look so parts aren't invisible. It is not the design. In the global stylesheet, declare the cascade layer at the top, above `@import "tailwindcss"` if that import exists, so Hallmark tokens and the page's CSS win:
   ```css
   @layer ui-sdk;
   ```
   A CSS Module loads too late to declare that layer. Keep the line in the global file.
5. **AGENTS.md.** Create the file if it isn't there, and add this note (wording matters; later sessions read it):

   > Before using ui-sdk, follow the ui-sdk skill if it's installed, and read `node_modules/@ui-sdk/react/docs/llms.txt`. To build a new component, also search `node_modules/@ui-sdk/react/docs/core.md` by heading, and read `rendering.md` next to it.

6. **The ui-sdk skill**, if this agent can install skills:
   ```bash
   npx skills add vercel-labs/ui --skill ui-sdk
   ```
   If it can't, the same skill ships at `node_modules/@ui-sdk/core/skills/ui-sdk/SKILL.md` after step 3. Read that before building or restyling a ui-sdk component. Don't restate its workflow here — the installed copy matches the installed version, and this file will go stale.

If the user only asked to set ui-sdk up, stop after these steps and ask what they'd like to build. If they already asked for a page, continue with Hallmark in the next section; don't stall on that question.

## 2. Set the direction with Hallmark

Do this **before** choosing a single component.

- **New page:** invoke Hallmark for the direction. Let it deal three themes (`scripts/theme.mjs deal`) and pick one with the user, or say **custom** to design from the prompt alone. Let `draw` lay out the anatomy from the page's job (a booking page books, a shop sells, a signup page signs up).
- **Existing page:** `/hallmark redesign <path>` for a full redesign (keeps content and wiring), add *keep the layout* for a restyle, or *polish* to fix only the tells. `/hallmark audit <path>` reports without changing anything.
- **App screens / dashboards** (not marketing pages): still take a theme or custom direction from Hallmark for type, colour, surfaces and spacing, then build with the project's own components.

Hallmark outputs self-contained HTML/CSS. In a React project, treat its output as the **spec**: translate its tokens (fonts, colours, radii, spacing, texture) into the project's Tailwind theme / CSS variables, and its sections into components. Record the decisions at the top of the main stylesheet or in `design.md` so later work stays consistent.

Build the **hero first, alone**, render it, look at it, then build the rest.

## 3. Choose components (only where they earn their place)

For each section, ask: *does motion or a richer component help the visitor do the page's job here?* If not, write plain, well-typed markup. Most sections should be static.

### Which library for what

| Need | First choice | Also consider |
|---|---|---|
| Logo wall, testimonial strip | Magic UI `marquee` | beUI `marquee` |
| Feature grid | Magic UI `bento-grid` | Kokonut cards |
| Stats that count up | Magic UI `number-ticker` | beUI `number` |
| Integration / data-flow diagram | Magic UI `animated-beam`, `orbiting-circles` | — |
| Product shots in a device frame | Magic UI `safari`, `iphone` | — |
| Subtle backgrounds | Magic UI `dot-pattern`, `grid-pattern`, `flickering-grid` | Kokonut `background-paths`, `beams-background`; beUI `shader-background` |
| Card highlight / border effect | Magic UI `magic-card`, `border-beam`, `shine-border` | beUI `tilt-card` |
| Text reveal on a heading | Magic UI `text-animate`, `blur-fade` | Kokonut `typewriter`, `matrix-text`; beUI `text-animation` |
| AI chat input, prompt box | Kokonut `ai-input*`, `v0-ai-chat` | beUI `prompt-input` |
| AI agent UI (messages, tool approval, citations, streaming, diffs) | beUI agents (`message`, `tool-approval`, `citations`, `streaming-response`, `file-diff`) | Kokonut `ai-tool-approval`, `ai-citations`, `ai-loading` |
| Tabs, switch, select, combobox, checkbox, accordion, field | UI SDK, each from its own entry (`@ui-sdk/react/tabs`, `select`, `field`, …) | beUI motion primitives, only if React is below 19 or the motion is the point |
| Modals, drawers, popovers, menus, tooltips | UI SDK `dialog`, `drawer`, `popover`, `menu`, `tooltip` | beUI `morphing-modal` when the morph itself is the signature effect |
| Command palette, OTP, file upload, slider, date picker | UI SDK `command`, `otp-input`, `file-input`, `slider`, `date-picker` | Kokonut `action-search-bar` if that visual is already the product's |
| Charts (line, bar, sparkline, sankey, candlestick) | beUI charts | the project's existing chart lib |
| Loaders | Kokonut `loader`, `ai-text-loading` | beUI `loader`, `loading-states` |
| Dock / app-style nav | Magic UI `dock` | beUI `dock` |
| Image-led gallery or portfolio with hover reveals | Obsidian `art-gallery`, `hover-img` | — |
| Parallax or split product/project showcase | Obsidian `parallax-slider`, `split-showcase` | — |
| Marquee the visitor can drag | Obsidian `draggable-marquee` | Magic UI `marquee` (non-draggable) |
| Editorial headline motion | Obsidian `flip-text`, `text-stream` (Text Reel) | Magic UI `text-animate` |
| Prismatic hero visual | Obsidian `v-prism` | — |
| Live status / session widgets | Obsidian `status-bars`, `active-sessions` | — |
| Dashboard layout starter | Obsidian `dashboard-shell`, `analytics-dashboard` | the project's own layout |

The shadcn slugs are a starting map, not a guarantee. **Confirm a slug exists before installing.** UI SDK slugs are the same idea: read `llms.txt` (or the component page it points at) and import only the parts that page lists. There is no root import from `@ui-sdk/react`. A missing control is built with the ui-sdk skill from `@ui-sdk/core`, not hand-rolled with `role=` on a `div`.

### Discover and verify
```bash
npx shadcn@latest search @kokonutui -q "input"
npx shadcn@latest search @magicui -q "marquee"
npx shadcn@latest search @beui -q "tabs"
npx shadcn@latest view "https://www.obsidianui.dev/r/hover-img.json"
npx shadcn@latest view @beui/tabs          # inspect files + deps before installing
```
Text indexes when the CLI isn't enough: `https://kokonutui.com/llms.txt`, `https://beui.dev/llms.txt`, `https://beui.dev/r` (JSON catalogue). Browse visually at `magicui.design/docs/components`, `kokonutui.com/docs`, `beui.dev/components/motion`, `obsidianui.dev/components`. Obsidian UI's registry slug can differ from its display name (Text Reel is `text-stream`, Hover Image is `hover-img`); take the slug from the component's docs URL.

### Motion budget
- **One** signature effect per page (usually in the hero). Everything else is quiet.
- At most one animated background on the whole page, and never behind body text.
- No more than one library's "style" per surface. Don't put a Magic UI shine border on a Kokonut card inside a beUI tilt card.
- Prefer UI SDK for things users *interact with*. It already does keyboard, focus, dismissal and ARIA, and it takes the page's styles, so it won't drag in another library's chrome. Prefer Magic UI for things users *look at*, Kokonut for *AI product surfaces* and loaders, Obsidian UI for *image-led, editorial* pages. Use beUI when the project can't run React 19, or when the motion itself is the component (a morphing modal, a tilt). Don't put a Magic UI shine border on a ui-sdk dialog.

### Obsidian UI is GSAP, not Motion
The other three libraries animate with Motion; Obsidian UI components use **GSAP** and ship a **CSS file** next to each component. So:
- Use Obsidian UI where its showpieces are clearly the best fit, not as a default. Two animation engines on one page means more JavaScript and two timing systems to keep in sync.
- Prefer at most one or two Obsidian components per page, ideally in sections that don't also use Motion components.
- Match its easing and durations to the rest of the page by editing the GSAP timelines in the installed file.
- Add reduced-motion handling yourself: check `window.matchMedia("(prefers-reduced-motion: reduce)")` or use `gsap.matchMedia()` and skip or shorten the timeline. Don't assume the component already does it.
- Kill GSAP timelines and ScrollTriggers on unmount (`gsap.context()` + `ctx.revert()`, or `useGSAP` from `@gsap/react`) so route changes don't leak animations.
- Move the colours and fonts in its CSS file onto the theme tokens from step 2.

## 4. Install and integrate

```bash
npx shadcn@latest add @magicui/marquee @beui/tabs @kokonutui/ai-input
```
- Install only what you'll use. Don't pass `--overwrite`; if the CLI asks to overwrite an existing file (common with generic names like `button`, `card`, `input`), stop and decide: keep the project's version, or install the new one under a different name.
- Files land in different places (`components/ui/`, `components/kokonutui/`, `components/motion/`, `components/block/` for Obsidian UI, which also adds a `.css` file per component and sometimes a `.jsx` file in a TypeScript project — type it or leave it, but don't let it break the build). Keep them there and import from where the CLI wrote them.
- **The code is now yours.** Edit it in place rather than wrapping it:
  - Replace hard-coded colours, fonts and radii with the theme tokens from step 2.
  - Delete demo content, placeholder avatars and fake data.
  - Tune durations and easing to match each other across libraries (pick one spring/ease and reuse it).
  - Wrap every animation in a reduced-motion guard (`useReducedMotion()` from `motion/react`, or `motion-safe:` / `motion-reduce:` Tailwind variants). Content must be fully visible with motion off.
- Client-only effects (canvas, WebGL, globe, confetti) need `"use client"` in Next.js App Router and should be lazy-loaded below the fold.
- UI SDK components are imported per entry, then styled from `data-slot` and state attributes (`data-open`, `data-pressed`, `data-disabled`) onto the theme tokens. Don't invent colours beside `web.css` and leave them. Change a part's element with `render`, not an `as` prop. Follow the installed ui-sdk skill for anything it already specifies, including when a custom primitive needs `"use client"`.

## 5. Audit in a real browser

Never hand off unrendered UI.

1. Run the dev server and look at the page yourself (browser tool or screenshots) at phone, tablet and desktop widths, in light and dark mode if the project has both.
2. Run Hallmark's linter. For a built page or a static export:
   ```bash
   node ~/.claude/skills/hallmark/scripts/sloplint.mjs <file-or-dir> --render
   ```
   For a live app, use `/hallmark audit <path>` or point Hallmark at the dev-server URL.
3. Fix every **Floor** failure (contrast, visible focus, reduced motion, invented facts, broken phone layout) — these never ship. Fix **Reflex** tells unless there is a stated reason to keep one. **Finish** items are optional polish.
4. Check what the libraries tend to break: layout shift when animations mount, horizontal scroll from marquees or beams on mobile, focus traps in morphing modals, keyboard access on custom selects and tabs, and bundle weight from heavy effects.
5. For important pages, get a fresh-context review (a subagent that hasn't seen the build) as Hallmark recommends.

## 6. Hand off

Tell the user briefly: the direction chosen (theme or custom), which components came from which library, anything that's still a placeholder waiting for real material, and the audit result. For redesigns, show before and after.

If this turn only set ui-sdk up, don't summarize a page that doesn't exist. Ask what they'd like to build.

---

## Anti-slop checklist (from Hallmark's gates, applies to every build)

Don't ship:
- an eyebrow label above every heading, or tracked all-caps labels everywhere
- one coloured or gradient word in the headline
- a hero that doesn't fit the first screen, or a page of empty frames
- a newsletter sign-up tacked onto the end of every page
- forms that say "thanks" without actually sending anything
- invented stats, testimonials, logos or reviews
- glow/gradient/particle effects stacked on top of each other
- motion with no reduced-motion fallback
- text below WCAG AA contrast, or controls without a visible focus state

## Licences

Kokonut UI, Magic UI, beUI (free tier), Obsidian UI and Hallmark are MIT. GSAP, which Obsidian UI depends on, is free to use under its own licence. UI SDK is Apache-2.0 and in beta; its APIs can change, so prefer the docs in `node_modules` over the website. Keep licence notices in copied source. beUI Pro, Magic UI Pro and Kokonut Pro are paid; only use them if the user has a licence and token.
