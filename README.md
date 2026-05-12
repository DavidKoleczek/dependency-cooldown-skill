# Dependency Cooldown Skill

An agent skill that configures in a project. A delay (typically 7 days) between when a package version is published and when it can be installed, mitigating most supply chain attacks.

The skill instructs an agent to survey the project's package managers and update bots, then add cooldown configuration at the package-manager layer (uv, pip, pnpm, npm, yarn, bun, deno, cargo) and/or the update-bot layer.

See [`SKILL.md`](./SKILL.md) for the full instructions.

## Installation

### Amplifier

Clone into your Amplifier skills directory:

```bash
git clone https://github.com/DavidKoleczek/dependency-cooldown-skill.git \
  ~/.amplifier/skills/dependency-cooldown-skill
```

The skill will be auto-discovered. Activate it with `load_skill(skill_name="dependency-cooldown-skill")` or let an agent invoke it when triggering conditions match.

### npx skills (vercel-labs/skills)

```bash
npx skills add DavidKoleczek/dependency-cooldown-skill
```

See [vercel-labs/skills](https://github.com/vercel-labs/skills) for tool details.

## License

MIT
