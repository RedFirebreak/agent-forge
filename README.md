# agent-forge

Skills, agents and workflows for coding agents, built while running real projects with them and shared as
they are actually used. Everything here is written for [Claude Code](https://code.claude.com) first. Skills
follow the open Agent Skills format, so they also work in other agents that read `SKILL.md`.

## What's in it

| Plugin | What it does |
|---|---|
| [gotcha](plugins/gotcha) | Keeps a symptom-first registry of the traps your project has hit: one entry per trap under a permanent ID like `HTTP-4`, an index you can match against what you're seeing, and a validator that keeps it all consistent. The next person, or the next session, recognises the problem from the symptom and skips the hours. |

More will follow as they prove themselves on real work.

## Install

In Claude Code, add this repository as a plugin marketplace once, then install what you want from it:

```text
/plugin marketplace add RedFirebreak/agent-forge
/plugin install gotcha@agent-forge
```

On Claude Code 2.1.275 or later, one line does both:
`/plugin install gotcha --marketplace RedFirebreak/agent-forge`.

The same from a shell:

```bash
claude plugin marketplace add RedFirebreak/agent-forge
claude plugin install gotcha@agent-forge
```

Plugins here carry no version number, so an update gives you the latest commit. Run
`/plugin marketplace update agent-forge`, or turn on auto-update for the marketplace under **Marketplaces**
in `/plugin`.

**Without the plugin system**, copy a skill's folder where your agent looks for skills. For Claude Code
that is `~/.claude/skills/` for every project, or `.claude/skills/` inside one repository:

```bash
git clone https://github.com/RedFirebreak/agent-forge
cp -r agent-forge/plugins/gotcha/skills/gotcha ~/.claude/skills/
```

## Layout

Each plugin is a folder under `plugins/`, in the standard Claude Code plugin layout:

```text
plugins/<name>/
├── .claude-plugin/plugin.json   the plugin's manifest
├── README.md                    what it is and how to use it
├── skills/<skill>/SKILL.md      skills, each with its scripts and assets beside it
└── agents/                      subagents, when the plugin has any
```

`.claude-plugin/marketplace.json` at the root lists every plugin; that file is what makes
`/plugin marketplace add RedFirebreak/agent-forge` work.

## Contributing

Issues and pull requests are welcome, especially a trap a skill missed or a check its validator should
have. A new plugin goes in its own folder under `plugins/` with an entry in `marketplace.json`, and
`claude plugin validate .` must pass.

## License

[MIT](LICENSE)
