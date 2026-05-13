# /clone-website — Competitor Cloning Mode

> **Want to replicate a site pixel-perfectly — animations, interactions, scroll effects and all?**
> This is a different workflow from `/build-website`. Use it when you have a live URL you want to clone.

---

## When to Use This vs. `/build-website`

| Goal | Use |
|------|-----|
| Build a new site from your own brief and Figma design | `/build-website` |
| Replicate a competitor or reference site exactly as-is | `/clone-website` (this guide) |

The clone workflow uses a dedicated template repo with a browser-automation pipeline that inspects the live site, extracts real CSS values, animations, and assets, then rebuilds every section in parallel.

---

## What It Does

```
You run:   /clone-website https://competitor.com
Claude:    Connects to the live site via browser automation
           → Takes screenshots at desktop + mobile
           → Extracts every design token via getComputedStyle()
           → Sweeps all interactions: scroll, click, hover
           → Writes spec files for every component
           → Dispatches parallel builder agents per section
           → Assembles the page and runs visual QA diff
```

**End result:**
- Every section rebuilt as a React component with exact CSS values (not approximations)
- All scroll behaviors, hover states, and animations cloned
- All images, videos, fonts, and SVGs downloaded
- Next.js + Tailwind v4 + shadcn/ui codebase you fully own

---

## Prerequisites (Different from `/build-website`)

### 1. Node.js 24+
The cloner template requires Node.js 24 (not 18).
```bash
node --version   # must be v24 or higher
```
Upgrade at https://nodejs.org if needed.

### 2. Claude Code — Opus 4.7
The pipeline uses parallel builder agents in git worktrees. Opus 4.7 is strongly recommended for best results — it handles the multi-agent orchestration and extraction quality significantly better than smaller models.

Start Claude Code with:
```bash
claude --chrome
```
The `--chrome` flag connects Claude to your Chrome browser, enabling it to interact with the live site (scroll, click, hover, take screenshots, read computed styles).

> Without `--chrome`, the clone command cannot work — it needs live browser automation to extract animations, hover states, and scroll-triggered behaviors.

### 3. GitHub CLI + Git (same as `/build-website`)
```bash
gh auth status
git --version
```

---

## Step-by-Step Setup

### Step 1: Clone the Template Repo

The `/clone-website` command comes pre-installed in a dedicated template. You set up one copy of this template per project (or fork it once and reuse).

```bash
git clone https://github.com/JCodesMore/ai-website-cloner-template.git my-clone
cd my-clone
npm install
```

This gives you a Next.js + Tailwind v4 + shadcn/ui scaffold that's already wired for the clone pipeline.

### Step 2: Start Claude Code with Chrome

```bash
claude --chrome
```

This opens Claude Code with Chrome browser automation enabled. Chrome must be installed on your machine. When prompted, allow Claude to control the browser.

> If you see "browser MCP not found" errors when running `/clone-website`, make sure you started Claude Code with the `--chrome` flag, not just `claude`.

### Step 3: Run the Clone Command

```
/clone-website https://www.competitor.com
```

You can also pass multiple pages:
```
/clone-website https://competitor.com https://competitor.com/pricing
```

Claude will process them in parallel, each isolated in its own `docs/research/<hostname>/` folder.

### Step 4: Let the Pipeline Run

The pipeline runs 5 phases automatically — no input needed unless something fails:

**Phase 1 — Reconnaissance**
- Full-page screenshots at 1440px and 390px
- Extracts fonts, colors, favicons, and global CSS
- Scrolls the entire page to discover every animation and behavior
- Clicks every interactive element (tabs, dropdowns, accordions)
- Hovers everything with hover states
- Tests at desktop, tablet, and mobile viewports

**Phase 2 — Foundation Build**
- Updates `layout.tsx` with the target's actual fonts
- Rewrites `globals.css` with the extracted color tokens and keyframes
- Creates TypeScript interfaces for the content structures
- Extracts all SVG icons as React components
- Downloads every image, video, and asset to `public/`

**Phase 3 — Component Spec & Dispatch**
- For each section of the page, writes a full spec file to `docs/research/components/`
- Every spec contains exact `getComputedStyle()` values, all states, all content verbatim
- Dispatches builder agents in parallel git worktrees — one per section/component
- While one agent builds, extraction continues on the next section

**Phase 4 — Assembly**
- Merges all worktree branches
- Wires components together in `app/page.tsx`
- Adds page-level behaviors: scroll snap, IntersectionObserver, Lenis smooth scroll
- Runs `npm run build` — must pass clean

**Phase 5 — Visual QA Diff**
- Takes side-by-side screenshots of original vs. clone
- Compares section by section at desktop and mobile
- Fixes any discrepancies found before declaring done

### Step 5: Customize and Deploy

After the clone is done:

```bash
# Push to your own GitHub repo
gh repo create my-site-clone --public --source=. --remote=origin --push

# Deploy to Vercel
npx vercel --prod --yes
```

---

## Tips for Best Results

**Use Opus 4.7 (not Sonnet)**
The clone pipeline dispatches multiple agents simultaneously and requires careful orchestration. Opus 4.7 handles this much better. Run `/fast` in Claude Code to switch to Opus if needed, or confirm the model in settings.

**Sites with heavy animations need more time**
Scroll-driven effects (parallax, sticky transforms, IntersectionObserver sequences) take extra extraction passes. Don't interrupt mid-run.

**Multiple URL cloning for multi-page sites**
Pass all pages at once: `/clone-website https://site.com https://site.com/about https://site.com/pricing`. Each gets its own research folder and the pipeline processes them in parallel.

**If Chrome MCP disconnects mid-run**
Reconnect by restarting Claude Code with `claude --chrome` and resume by describing what phase you were in. The spec files already written and assets already downloaded are preserved.

**Smooth scroll libraries (Lenis, Locomotive Scroll)**
The pipeline checks for these automatically. If the target uses Lenis, it will install it and wire it up. The clone should feel identical to scroll — not just look identical.

---

## What Gets Cloned vs. What Doesn't

| Cloned | Not Cloned |
|--------|------------|
| Visual layout and all CSS | Backend / database |
| Animations and scroll behaviors | Authentication |
| Hover, click, focus interactions | Real-time features (live data, chat) |
| Responsive breakpoint behavior | SEO optimization |
| All fonts, images, videos, SVGs | Third-party integrations (analytics, payments) |
| Exact colors, typography, spacing | Server-side rendering logic |

---

## Important: Ethical Use

This tool clones the **structure and visual design** of a site. Before using it:

- **Do not clone sites for impersonation or phishing** — this is illegal and the template explicitly prohibits it.
- **Do not pass off someone else's brand assets (logos, copy) as your own.**
- **Check the target site's ToS** — some sites prohibit scraping or reproduction.
- **Legitimate use cases:** rebuilding a site you own (lost source code), migrating from Webflow/Squarespace to Next.js, learning how a specific layout or animation was built, building your own design inspired by a reference.

---

## Troubleshooting

**`/clone-website` command not found**
→ You need to be inside the cloned template repo (`cd my-clone`) and in a new Claude Code session started inside that directory.

**"Browser MCP not detected"**
→ You started Claude Code without `--chrome`. Close and restart: `claude --chrome`.

**Chrome won't open / permission denied**
→ On macOS: System Preferences → Privacy & Security → Accessibility → allow Terminal/Claude.

**Builder agent fails TypeScript check**
→ The pipeline requires `npx tsc --noEmit` to pass before each merge. If a builder fails, Claude will fix it before moving on.

**Clone looks right in screenshots but animations don't work**
→ The site likely uses a smooth scroll library (Lenis). Check if Claude detected and installed it during Phase 2. If not, ask: "Did you install Lenis or any smooth scroll library?"

**Images show as broken in the clone**
→ Some images are behind CDN auth. Claude will fall back to placeholder images for those — replace them manually with the originals.
