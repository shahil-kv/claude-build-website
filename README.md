# /build-website — Claude Code Skill

> **One command. Prompt → Code → GitHub → Vercel → Figma. Done.**

`/build-website` is a custom [Claude Code](https://claude.ai/code) slash command that runs a complete design-to-deployment pipeline for a new Next.js landing page. Give it a brief description of your site and a Figma design system URL — it handles everything else.

---

## What It Does

```
You type:  /build-website
Claude:    Asks 2 questions → analyzes your Figma design system
           → refines your brief → generates production code
           → pushes to GitHub → deploys to Vercel
           → captures the live site into Figma
           → sets up a Figma → code sync pipeline
```

**End result every time:**
- A full Next.js 15 + Tailwind v4 + shadcn/ui landing page
- All brand colors wired to CSS variables (zero hardcoded hex)
- Live on Vercel with a production URL
- Pixel-perfect capture in your Figma file
- A sync script so Figma color changes flow back into code with one command

---

## Install

```bash
mkdir -p ~/.claude/commands && curl -o ~/.claude/commands/build-website.md \
  https://raw.githubusercontent.com/shahil-kv/claude-build-website/main/build-website.md
```

Then **restart Claude Code** (or open a new session). Type `/build` and you'll see it in the autocomplete.

> **Project-level install** (for teams — no curl needed per person):
> ```bash
> mkdir -p .claude/commands && curl -o .claude/commands/build-website.md \
>   https://raw.githubusercontent.com/shahil-kv/claude-build-website/main/build-website.md
> git add .claude/commands/build-website.md && git commit -m "add build-website skill"
> ```
> Anyone who clones the repo and opens Claude Code gets `/build-website` automatically.

---

## Prerequisites

You need **all five** of these before the command will work end-to-end. Missing any one will cause it to fail partway through.

### 1. Claude Code with a paid plan
- Download: https://claude.ai/code
- A Pro or higher Anthropic subscription is required (free tier does not support MCP plugins)
- The command uses Claude's tool-use capabilities — it won't work in the regular chat UI

### 2. Figma MCP Server connected
- Open Claude Code → Settings → MCP Servers → Add Server
- Type: `Streamable HTTP`
- URL: `https://mcp.figma.com/mcp`
- ID: `figma`
- You must be logged into Figma in your browser when you run it

> ⚠️ **IMPORTANT — Figma MCP Rate Limits:**
> - **Starter / Viewer seats**: **6 tool calls per month**. This skill uses ~4–8 calls per run. You may hit the limit on your second site.
> - **Dev or Full seat** on Professional / Organization / Enterprise: per-minute limits (same as Figma REST API Tier 1). This is the tier you want for regular use.
> - Check your seat type at figma.com → Settings → Plan & billing
> - The write-to-canvas feature (capturing your site into Figma) is **currently free in beta** but will become a paid feature in the future.

### 3. Vercel MCP Server connected
- Open Claude Code → Settings → MCP Servers → Add Server
- Type: `Streamable HTTP`
- URL: `https://mcp.vercel.com/mcp`
- ID: `vercel`
- You must have a Vercel account: https://vercel.com/signup (free tier works)

> **Tip:** With Vercel MCP connected, Step 6 can use the `deploy_to_vercel` MCP tool directly as an alternative to the Vercel CLI — useful if you prefer not to install the CLI globally.

### 4. GitHub CLI (`gh`) installed and authenticated
```bash
# Install (macOS)
brew install gh

# Install (Windows)
winget install GitHub.cli

# Authenticate
gh auth login
# → Choose: GitHub.com → HTTPS → Login with a web browser
```
The skill uses `gh repo create` to create your GitHub repo. The GitHub MCP alternative often lacks the right token scope, so `gh` CLI is the reliable path.

> Verify auth before starting: `gh auth status`

### 5. Node.js 18+ and Git
```bash
node --version   # should be v18 or higher
git --version    # any recent version
```

---

## Want to Clone a Competitor's Site Instead?

If you already have a live URL you want to replicate pixel-perfectly — including animations, scroll effects, hover states, and interactions — use the **`/clone-website`** workflow instead.

```
/clone-website https://www.competitor.com
```

This is a separate pipeline that uses browser automation to inspect the live site, extract exact CSS values via `getComputedStyle()`, and dispatch parallel builder agents to reconstruct every section.

**Read the full guide:** [clone-website.md](./clone-website.md)

| | `/build-website` | `/clone-website` |
|---|---|---|
| **Starting point** | Your brief + Figma design | A live URL |
| **Output** | New site from your design | Pixel-perfect replica |
| **Animations** | What you design | Cloned exactly from source |
| **Browser needed** | No | Yes (`claude --chrome`) |
| **Node.js required** | 18+ | 24+ |
| **Best model** | Sonnet | Opus 4.7 |

---

## Preflight Check

Run these before starting to avoid failures mid-way:

```bash
gh auth status          # must show "Logged in to github.com"
node --version          # must be v18+
git --version           # any version
npx vercel whoami       # must show your Vercel username
```

If any command fails, fix it before running `/build-website`.

---

## Optional but Recommended

### Figma Personal Access Token (for Figma → code sync)

The sync pipeline (Step 8) calls the Figma REST API to read your Local Variables and patch your CSS. It needs a personal access token stored as a GitHub secret.

**Get your token:**
1. figma.com → top-left menu → Settings
2. Security tab → Personal access tokens → Generate new token
3. Name it "Claude Sync" — copy the value

**Add it as a GitHub secret** (after your repo is created in Step 5):
```
github.com/[you]/[your-site]/settings/secrets/actions/new
Name:  FIGMA_TOKEN
Value: [your token]
```

Without this token, the sync GitHub Action will fail. The rest of the workflow (code + deploy + Figma capture) works fine without it.

### Vercel CLI
```bash
npm i -g vercel
vercel login
```
The skill runs `npx vercel` which downloads it on demand, but having it installed globally speeds things up.

---

## Usage

```
/build-website
```

Claude will ask you:

**Question 1 — What is this website for?**
Tell it: brand name, product/niche, target audience, tone (minimal, bold, dark, playful), which sections you want, any reference sites.

Example:
> "Unlink — a screen time blocking app for people who want to reclaim focus. Dark/minimal, green accent. Sections: Nav, Hero, Problem stats, How it works, Features, Testimonials, Pricing, CTA, Footer."

**Question 2 — Figma styling guide URL**
Paste your Figma design system file URL. If you don't have one, just say "use the default" and it'll use the Shadcn UI community design system.

After that, Claude presents a brief for your approval, then builds everything automatically.

---

## How the Sync Works

After your site is live, your brand colors live as CSS variables in `app/globals.css`:

```css
:root {
  --brand-bg:        #F5F2ED;
  --brand-accent:    #C4A96A;
  /* ... */
}
```

These map 1:1 to Figma Local Variables. To update them:

1. Edit your Local Variables in Figma (Local Variables panel → edit values)
2. Tell Claude: **"sync and deploy"**
   - Claude runs `npm run sync-figma` (reads Figma REST API → patches globals.css)
   - Commits and pushes
   - Vercel automatically redeploys

Or trigger the GitHub Action manually:
```
github.com/[you]/[site]/actions → "Sync Figma Design Tokens" → Run workflow
```

> **Why not auto-sync on every Figma change?**
> Auto-webhooks create a commit for every cursor move in Figma — your git history becomes noise. The deliberate pattern means one clean commit per intentional design update.

---

## Tech Stack Generated

Every site built with this skill uses:

| Layer | Choice | Why |
|-------|--------|-----|
| Framework | Next.js 15 App Router | Latest stable, Vercel-native |
| Styling | Tailwind CSS v4 | Zero config, utility-first |
| Components | shadcn/ui | Accessible, copy-paste, design-system-ready |
| Fonts | Google Fonts via `next/font` | Zero layout shift |
| Colors | CSS custom properties | Single source of truth, sync-friendly |
| Icons | Lucide React | Consistent, tree-shakeable |
| Deployment | Vercel | Zero-config Next.js hosting |
| Design | Figma | Source of truth for tokens |

---

## Troubleshooting

**`/build-website` doesn't appear in autocomplete**
→ You need to start a **new Claude Code session** after installing. The command list is loaded at session start.

**Figma MCP error: "rate limit exceeded"**
→ You've hit the 6 calls/month limit on the Starter plan. Upgrade to a Dev or Full seat on a paid Figma plan, or wait until next month.

**`gh: command not found`**
→ Install GitHub CLI: `brew install gh` (macOS) or see https://cli.github.com/

**`gh repo create` fails with 403**
→ Run `gh auth login` and re-authenticate. Make sure you choose "repo" scope.

**Vercel deployment fails: "token not valid"**
→ Run `npx vercel login` in the project directory, then retry. Or use the Vercel MCP `deploy_to_vercel` tool as an alternative.

**`npm run build` fails with TypeScript errors**
→ Fix all TypeScript errors before proceeding — do not skip build verification. Common causes: missing type imports, unresolved shadcn components. Run `npm run build` again after each fix.

**Sync script fails: "FIGMA_TOKEN is not set"**
→ Add `FIGMA_TOKEN` as a GitHub secret (see Prerequisites → Figma Personal Access Token above).

**Sync script runs but CSS doesn't change**
→ Your Figma Local Variables are named differently from what the script expects. The script looks for variables named `brand/bg`, `brand/dark`, `brand/accent`, `brand/secondary`, `brand/border`, `brand/surface`. Rename them in Figma to match, or edit the script's token map.

**Build error: `Cannot find module 'tw-animate-css'`**
→ Run `npm install tw-animate-css` in your project.

**`npx vercel whoami` fails before starting**
→ Run `npx vercel login` first, then retry the preflight check.

---

## Limitations

- Generates landing pages (marketing sites). Not suited for dashboards, e-commerce checkout flows, or apps with auth.
- The Figma sync only patches CSS color/spacing variables — it does not sync component structure, animations, or layout changes.
- Requires the five prerequisites above. If your org blocks GitHub CLI or Vercel, the deploy steps will need manual intervention.
- Image assets in Figma are not synced to code (only color tokens and spacing values).
- The Figma capture in Step 7 uses `get_screenshot` — it captures a visual snapshot, not an editable component tree.
- Does **not** clone existing sites. For that, use the `/clone-website` workflow — see [clone-website.md](./clone-website.md).

---

## Contributing

Found a bug or want to improve the workflow? Open an issue or PR. The entire skill is one markdown file — `build-website.md`. Edit it and the change is live for anyone who re-runs the curl install.

---

## License

MIT — use it, fork it, share it.

---

*Built by Shahil · Powered by [Claude Code](https://claude.ai/code)*
