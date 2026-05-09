---
description: "Full design-to-deployment workflow: prompt → Next.js code → GitHub → Vercel → Figma capture → sync script. Asks for site description and Figma styling guide URL, then builds and ships everything."
---

# /build-website

Turn a site idea into a fully deployed Next.js site, captured in Figma, with a live Figma→code sync pipeline. Run this command to start a new site from scratch.

## Step 0: Gather Requirements

Before writing any code, ask the user **two questions** (both required):

1. **What is this website for?**
   Collect: the brand/product name, niche, target audience, tone (minimal, bold, dark, playful, etc.), must-have sections, and any reference sites.

2. **What is your Figma styling guide URL?**
   This is the Figma file that contains the design system — color tokens, typography, components. Parse the `fileKey` and `nodeId` from the URL.
   - If the user does not have one yet, use the Shadcn UI community design system as default:
     `https://www.figma.com/design/0fAHdmB6vJsyoGbSFmjm61/-shadcn-ui---Design-System--Community-?node-id=2-287`

Do NOT proceed to Step 1 until both answers are collected.

---

## Step 1: Analyze the Figma Design System

Use `get_design_context` (Figma MCP) on the styling guide URL to extract:
- **Color palette**: background, foreground, accent, secondary, border, surface, destructive
- **Typography**: heading font family + weights, body font family + weights
- **Spacing scale**: base unit, border-radius values
- **Component patterns**: button styles, card patterns, nav style

Map what you find into a brand token table like:

| Token | CSS Variable | Value |
|-------|-------------|-------|
| Background | `--brand-bg` | #F5F2ED |
| Dark | `--brand-dark` | #1A1714 |
| Accent | `--brand-accent` | #C4A96A |

If `get_design_context` hits a rate limit, ask the user to share a screenshot of the Figma file's color and font styles.

---

## Step 2: Prompt Refinement

Before writing code, produce a refined site brief:

```
SITE BRIEF
----------
Brand name: [name]
Tagline: [one-line positioning statement]
Niche/audience: [who this is for]
Tone: [3 adjectives]
Sections (in order):
  1. Nav — [logo + links + CTA]
  2. Hero — [main headline + sub + CTA + image treatment]
  3. [section 3 name] — [purpose]
  ...
  N. Footer

Typography:
  Heading: [font] [weights]
  Body: [font] [weights]

CSS variables mapped from Figma:
  --brand-bg, --brand-dark, --brand-accent, --brand-secondary, --brand-border, --brand-surface
```

Show this brief to the user and ask: "Does this look right? Anything to adjust?"
Only proceed after user confirms.

---

## Step 3: Generate the Code

### File: `app/layout.tsx`

- Import heading + body fonts from `next/font/google`
- Set CSS variable names matching the fonts (e.g. `--font-display`, `--font-body`)
- Export metadata with brand name and description
- Apply both font variables to `<html>` with `antialiased`

### File: `app/globals.css`

Structure:
```css
@import "tailwindcss";
/* optional animation keyframes here */
@import "tw-animate-css";
@import "shadcn/tailwind.css";

/* BRAND TOKENS — map 1:1 to Figma Local Variables */
:root {
  /* Figma Variables to create (Local Variables panel):
       brand/bg        → [hex]
       brand/dark      → [hex]
       brand/accent    → [hex]
       brand/secondary → [hex]
       brand/border    → [hex]
       brand/surface   → [hex]
  */
  --brand-bg:        [hex];
  --brand-dark:      [hex];
  --brand-accent:    [hex];
  --brand-secondary: [hex];
  --brand-border:    [hex];
  --brand-surface:   [hex];
}

@custom-variant dark (&:is(.dark *));

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --font-sans: var(--font-body);
  --font-heading: var(--font-display);
  /* ... shadcn theme tokens ... */
}

:root { /* shadcn light mode tokens */ }
.dark { /* shadcn dark mode tokens */ }

@layer base {
  * { @apply border-border outline-ring/50; }
  body { @apply bg-background text-foreground; }
  html { @apply font-sans; }
}
```

**Critical rules for CSS:**
- ALL brand colors must use `var(--brand-*)` — zero hardcoded hex in components
- Shadows, borders, and backgrounds all reference CSS variables
- Add animation keyframes for any motion effects (marquee, fadeUp, etc.) at the top

### File: `app/page.tsx`

Build a complete, production-quality landing page with at minimum:

1. **Nav** — sticky, logo left + links center/right + CTA button. Uses `var(--brand-bg)` background, `var(--brand-dark)` text, `var(--brand-border)` border-bottom.

2. **Hero** — full-viewport. Headline uses heading font. CTA button uses `var(--brand-accent)` background. If visual interest needed, add a marquee ticker or image overlay.

3. **Social Proof / Logos strip** (if consumer product)

4. **Problem / Why Us** section — dark background `var(--brand-dark)`, alternating stats with animated counters using IntersectionObserver

5. **How It Works / Features** — 3-column cards. Cards use `var(--brand-surface)` background. Hover state adds `var(--brand-accent)` accent border.

6. **Testimonials** — Use `https://i.pravatar.cc/48?img=[N]` for avatars (N=1-70). Star ratings in `var(--brand-accent)`.

7. **Pricing** (if applicable) — 3 tiers, highlighted middle tier uses `var(--brand-accent)` border.

8. **CTA Section** — final conversion moment.

9. **Footer** — dark background, columns, copyright.

**Quality rules:**
- Mobile-first responsive (sm/md/lg breakpoints)
- Hover transitions on all interactive elements (200ms ease)
- Real copy (no "Lorem ipsum") — invent compelling brand copy from the brief
- Real Unsplash images: `https://images.unsplash.com/photo-[id]?w=1200&q=80`
- No hardcoded colors — every color via `var(--brand-*)` or shadcn tokens

---

## Step 4: Scaffold the Project

```bash
# Create Next.js app
npx create-next-app@latest [site-name] \
  --typescript --tailwind --eslint \
  --app --no-src-dir --no-import-alias

cd [site-name]

# Install dependencies
npm install lucide-react clsx tailwind-merge class-variance-authority
npm install tw-animate-css
npx shadcn@latest init -d
```

Write the three files from Step 3, then verify it builds:
```bash
npm run build
```

Fix any TypeScript or build errors before proceeding.

---

## Step 5: Push to GitHub

```bash
# Initialize git
git init
git add -A
git commit -m "feat: initial [site-name] landing page"

# Create GitHub repo and push using gh CLI
gh repo create [site-name] --public --source=. --remote=origin --push
```

If `gh` is not authenticated, tell the user: `! gh auth login`

---

## Step 6: Deploy to Vercel

```bash
# Link and deploy
npx vercel --yes
npx vercel --prod --yes
```

If the CLI is not installed: `npm i -g vercel`
If not authenticated: `npx vercel login`

Save the production URL — you'll need it for Step 7.

---

## Step 7: Capture to Figma

Use `generate_figma_design` (Figma MCP) with the production Vercel URL to create a pixel-perfect capture in the user's Figma file.

Target file: use the same Figma file as the styling guide, or create a new one.

After capture, confirm the node ID of the captured frame — save it for future reference.

---

## Step 8: Set Up Figma → Code Sync

### Script: `scripts/sync-figma-tokens.js`

Create this script that reads Figma Local Variables and patches `app/globals.css`:

```js
const FIGMA_FILE_KEY = "REPLACE_WITH_ACTUAL_FILE_KEY";
const FIGMA_TOKEN = process.env.FIGMA_TOKEN;

async function fetchFigma(path) {
  const res = await fetch(`https://api.figma.com/v1${path}`, {
    headers: { "X-Figma-Token": FIGMA_TOKEN },
  });
  if (!res.ok) throw new Error(`Figma API error: ${res.status} ${await res.text()}`);
  return res.json();
}

function hexFromRgba({ r, g, b }) {
  return "#" + [r, g, b].map(c => Math.round(c * 255).toString(16).padStart(2, "0")).join("");
}

async function main() {
  const data = await fetchFigma(`/files/${FIGMA_FILE_KEY}/variables/local`);
  const variables = Object.values(data.meta?.variables ?? {});
  const modes = Object.values(data.meta?.variableCollections ?? {})
    .flatMap(c => c.modes);
  const defaultMode = modes[0]?.modeId;

  const tokens = {
    "--brand-bg":        null,
    "--brand-dark":      null,
    "--brand-accent":    null,
    "--brand-secondary": null,
    "--brand-border":    null,
    "--brand-surface":   null,
  };

  for (const v of variables) {
    const cssName = "--" + v.name.replace(/\//g, "-").replace(/\s+/g, "-").toLowerCase();
    if (!(cssName in tokens)) continue;
    const val = v.valuesByMode?.[defaultMode];
    if (!val) continue;
    if (v.resolvedType === "COLOR" && val.r !== undefined) {
      tokens[cssName] = hexFromRgba(val);
    } else if (v.resolvedType === "FLOAT") {
      tokens[cssName] = `${val}px`;
    }
  }

  const fs = await import("fs");
  let css = fs.readFileSync("app/globals.css", "utf8");
  for (const [key, value] of Object.entries(tokens)) {
    if (!value) continue;
    css = css.replace(
      new RegExp(`(${key.replace(/[-]/g, "\\-")}\\s*:\\s*)([^;]+)(;)`),
      `$1${value}$3`
    );
  }
  fs.writeFileSync("app/globals.css", css);
  console.log("Synced tokens:", Object.fromEntries(Object.entries(tokens).filter(([,v]) => v)));
}

main().catch(console.error);
```

Add to `package.json` scripts:
```json
"sync-figma": "node scripts/sync-figma-tokens.js"
```

### GitHub Action: `.github/workflows/sync-figma.yml`

```yaml
name: Sync Figma Design Tokens
on:
  workflow_dispatch:
    inputs:
      message:
        description: "Commit message (optional)"
        required: false
        default: "chore: sync design tokens from Figma"

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
      - name: Sync tokens
        env:
          FIGMA_TOKEN: ${{ secrets.FIGMA_TOKEN }}
        run: node scripts/sync-figma-tokens.js
      - name: Commit if changed
        run: |
          git config user.name "Figma Sync Bot"
          git config user.email "bot@figmasync.local"
          git diff --quiet app/globals.css || (
            git add app/globals.css &&
            git commit -m "${{ github.event.inputs.message }}"
          )
      - uses: ad-m/github-push-action@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Tell the user to add the secret:
```
Go to: github.com/[username]/[repo-name]/settings/secrets/actions/new
Name:  FIGMA_TOKEN
Value: your Figma personal access token (figma.com → Settings → Security → Personal access tokens)
```

---

## Step 9: Confirm Handoff

Print a summary:

```
✓ Site built:     [site-name]
✓ GitHub:         github.com/[username]/[site-name]
✓ Live URL:       https://[site-name].vercel.app
✓ Figma capture:  [figma-url]
✓ Sync script:    scripts/sync-figma-tokens.js (npm run sync-figma)
✓ GitHub Action:  .github/workflows/sync-figma.yml (trigger manually)

HOW TO UPDATE FROM FIGMA:
1. Edit your Figma Local Variables (brand/bg, brand/dark, etc.)
2. Tell me: "sync and deploy" — I'll run the sync and push to GitHub
   OR trigger manually: GitHub → Actions → "Sync Figma Design Tokens" → Run workflow
```

---

## Notes and Decisions

**Why deliberate sync, not auto-webhook?**
Auto-syncing on every Figma change creates noise commits. The deliberate pattern ("tell me when you're ready") keeps the git history clean — one commit per intentional design update, not one per cursor move.

**Why CSS variables as the sync bridge?**
Figma Local Variables → REST API → CSS custom properties is the simplest reliable path. Every color reference in JSX reads from `var(--brand-*)`, so one CSS change updates the entire site.

**Why `gh` CLI for GitHub, not the GitHub MCP?**
The GitHub MCP PAT often lacks `repo:create` scope. The `gh` CLI uses the user's existing OAuth token which always has the right permissions.

**Why `workflow_dispatch` for the GitHub Action?**
Gives the user deliberate control. They can also just say "sync and deploy" in this chat — I'll run the sync script locally, commit, and push, which triggers Vercel's automatic build.
