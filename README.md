# polished-ui

Agent skill for building or upgrading web UI that looks custom, not generated. Hallmark sets the design direction. Kokonut UI, Magic UI, beUI, and Obsidian UI only supply parts.

## Install

From any project, or globally so every agent session can use it:

```bash
npx skills add AmoDinho/polished-ui -g -y
```

Install into the current project only (omit `-g`):

```bash
npx skills add AmoDinho/polished-ui -y
```

Target one agent if you don't want it everywhere:

```bash
npx skills add AmoDinho/polished-ui -g -a opencode -a claude-code -y
```

List it without installing:

```bash
npx skills add AmoDinho/polished-ui --list
```

Requires Node.js 22.20 or later for the skills CLI. The skill itself also expects Hallmark (`npx skills add nutlope/hallmark -g`) when a page needs a design direction or a browser audit.

## Layout

`SKILL.md` sits at the repo root so `npx skills add` discovers it. The skill name is `polished-ui`.
