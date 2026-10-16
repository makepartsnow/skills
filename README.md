# Make Parts Now agent skills: 3D printing and CNC machining quotes for coding agents

[![skills.sh](https://skills.sh/b/makepartsnow/skills)](https://skills.sh/makepartsnow/skills)

Agent skills from [MakePartsNow.com](https://makepartsnow.com), a US 3D printing and CNC machining service. They teach a coding agent (Claude Code, Codex, Cursor, Gemini CLI, OpenCode and the other agents the [skills CLI](https://github.com/vercel-labs/skills) supports) to take a part from your project to a real quote and order:

- export the model from code CAD (CadQuery, build123d, OpenSCAD, FreeCAD) or your CAD tool,
- quote it in SLS PA12 nylon, TPU, FDM plastic, 6061 aluminum or C360 brass, with the price, ship date and arrival date for each production speed,
- compare quantities (unit prices from 1 to 1000 pieces) and change the material, finish or speed,
- open a checkout link you pay in your browser.

## Install

```bash
npx skills add makepartsnow/skills
```

Add `-g` to install for all your projects, or `-a claude-code` (or another agent) to pick the agent. Then ask your agent "Quote 25 of `cad/bracket.step` in black PA12 nylon" or "What would 10, 50 and 100 of this enclosure cost?"

## Skills

| Skill | What it does |
|---|---|
| [`make-parts-now`](skills/make-parts-now/SKILL.md) | CAD export, quote, quantity comparison, checkout and order status through the [Make Parts Now REST API](https://makepartsnow.com/developers), or through the Make Parts Now MCP tools when a plugin already provides them |

The skill is instructions only. It calls the API with `curl` and adds no code, scripts or hooks to your machine.

## What you should know

- **Account.** Quoting through the REST API needs a free agent account. The skill asks before creating one, with your name and email and your acceptance of the [terms](https://makepartsnow.com/legal/terms), and tells you to keep the key in `MPN_API_KEY` (never in your repo). Order notices go to that email, and you can sign in on makepartsnow.com to see your orders.
- **Network.** Only `https://makepartsnow.com` (`/api/agent/v1`).
- **Your files.** The agent uploads only the CAD file you ask it to quote, after confirming it isn't ITAR, EAR or CUI controlled. Files that are never ordered are deleted; see the [privacy policy](https://makepartsnow.com/legal/privacy).
- **Money.** The agent never pays. A checkout link opens Stripe Checkout on makepartsnow.com, where you review the order, enter the shipping address and pay. Prices are in USD before sales tax; shipping is within the United States.

## Prefer a plugin?

The same service is available as a plugin with the MCP server built in: [Claude](https://github.com/makepartsnow/makepartsnow-claude), [Cursor](https://github.com/makepartsnow/makepartsnow-cursor), [Gemini CLI](https://github.com/makepartsnow/makepartsnow-gemini), [Grok Build](https://github.com/makepartsnow/makepartsnow-grok), [OpenClaw](https://github.com/makepartsnow/makepartsnow-openclaw) and [Hermes Agent](https://github.com/makepartsnow/makepartsnow-hermes). See [makepartsnow.com/developers](https://makepartsnow.com/developers).

## Support

Orders and questions: orders@makepartsnow.com or [makepartsnow.com/contact](https://makepartsnow.com/contact). Problems with a skill: open an issue here; report security problems privately to orders@makepartsnow.com.

## License

MIT, for the files in this repo. The Make Parts Now service is covered by its [terms](https://makepartsnow.com/legal/terms).
