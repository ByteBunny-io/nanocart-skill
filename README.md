# NanoCart AI skill

Teach your AI coding agent to integrate [NanoCart](https://nanocart.io) — embed the
cart widget, wire buy buttons, theme it, add email signup, and automate your catalog.

## Install — Claude Code

**Marketplace (recommended):**
```
/plugin marketplace add ByteBunny-io/nanocart-skill
/plugin install nanocart@nanocart-skill
```

**Manual:** download https://cdn.nanocart.io/ai/nanocart-skill.zip and copy the
`skills/nanocart/` folder into your project's `.claude/skills/` (or `~/.claude/skills/`
for all projects).

Then just ask: *"add nanocart to my site"* — or invoke `/nanocart` directly.

## Install — Codex / Cursor / other agents

Copy `AGENTS.md` from this package into your project root (or merge its contents into
your existing AGENTS.md / rules file).

## Credentials

Copy `env.example` to `.env`. Your **Store ID** is public (script tag). Your
**API key** is secret — server-side use only, never in website code.

## Example prompts

- "Add NanoCart to my site and put Buy Now buttons on the three product cards"
- "Theme the cart to match my site's colors"
- "Create products in my NanoCart store from the images in ./products"
- "Add a newsletter signup band above the footer"

## Docs

- Support docs: https://docs.nanocart.io (AI-readable: /llms.txt, /llms-full.txt)
- API reference: https://nanocart.io/docs

v1.0.0 · © 2026 NanoCart · a ByteBunny, LLC company
