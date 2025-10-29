# Claude Skills Marketplace

A marketplace of Agent Skills plugins for Claude Code that extend Claude's capabilities with specialized workflows and tools.

## Plugins Available

### 1. Project Management Plugin (`project-management-plugin`)

Project lifecycle management tools for git workflows, code review, and system architecture.

**Skills included:**
- **Git Commit Helper** - Generate descriptive commit messages following conventional commit format
- **PR Review Helper** - Create pull requests with interactive review and comprehensive descriptions

**Commands:**
- **/ggc** - Create a git commit with documentation checks and pre-commit hooks
- **/system-architect** - Design new features, modules, or systems with comprehensive architectural analysis

**Agents:**
- **code-reviewer** - Comprehensive code review agent for staged git changes

**Use when:** Working with git commits, pull requests, code reviews, or designing new system features.

### 2. Meta Plugin (`meta-plugin`)

Tools for creating and managing Claude Code skills and plugins.

**Skills included:**
- **Skill Creator** - Guide for creating effective Agent Skills with bundled resources and proper structure

**Use when:** Creating or updating Agent Skills and plugins.

### 3. Homelab Plugin (`homelab-plugin`)

Skills for managing homelab infrastructure including remote server management.

**Skills included:**
- **TrueNAS Docker Operations** - Interact with Docker containers on TrueNAS via SSH with helper scripts for complex operations

**Use when:** Working with containers on remote TrueNAS servers, querying databases, or executing commands.

## Installation

### From GitHub (Recommended)

1. Add this repository as a marketplace:
```bash
/plugin marketplace add thurstonsand/claude-skills
```

2. Install the plugins you want:
```bash
# Install all plugins
/plugin install project-management-plugin@claude-skills-marketplace
/plugin install meta-plugin@claude-skills-marketplace
/plugin install homelab-plugin@claude-skills-marketplace

# Or install only specific plugins you need
/plugin install project-management-plugin@claude-skills-marketplace
```

3. Restart Claude Code to activate the skills.

### Local Development

For testing or contributing:

1. Clone this repository
2. Add as a local marketplace:
```bash
/plugin marketplace add /path/to/claude-skills
```

3. Install plugins:
```bash
/plugin install project-management-plugin@claude-skills-marketplace
/plugin install meta-plugin@claude-skills-marketplace
/plugin install homelab-plugin@claude-skills-marketplace
```

## Repository Structure

```
claude-skills/                    # Marketplace root
├── .claude-plugin/
│   └── marketplace.json          # Marketplace configuration
├── project-management-plugin/    # Project lifecycle plugin
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── commands/
│   │   ├── ggc.md
│   │   └── system-architect.md
│   ├── agents/
│   │   └── code-reviewer.md
│   └── skills/
│       ├── git-commit-helper/
│       │   └── SKILL.md
│       └── pr-review-helper/
│           ├── SKILL.md
│           └── scripts/
├── meta-plugin/                  # Skill creation plugin
│   ├── .claude-plugin/
│   │   └── plugin.json
│   └── skills/
│       └── skill-creator/
│           ├── SKILL.md
│           ├── scripts/
│           └── LICENSE.txt
├── homelab-plugin/               # Homelab infrastructure plugin
│   ├── .claude-plugin/
│   │   └── plugin.json
│   └── skills/
│       └── truenas-docker-ops/
│           ├── SKILL.md
│           ├── scripts/
│           └── references/
└── README.md
```

## Usage

Once installed, Claude will automatically use these skills based on task context:

**Project Management Plugin:**
- Ask Claude to help with commit messages → uses `git-commit-helper` skill
- Ask to create a pull request → uses `pr-review-helper` skill
- Use `/ggc` to create a git commit with documentation checks
- Use `/system-architect` to design new features or systems
- Request code review → uses `code-reviewer` agent

**Meta Plugin:**
- Ask to create a new skill → uses `skill-creator`

**Homelab Plugin:**
- Ask about Docker containers on TrueNAS → uses `truenas-docker-ops`

You can mix and match plugins based on your needs - install only the ones you want to use.

## Contributing

Contributions are welcome! To add or improve skills:

1. Fork this repository
2. Make your changes following the [Agent Skills documentation](https://docs.claude.com/en/docs/claude-code/skills)
3. Test locally using the local development installation method
4. Submit a pull request

## License

See individual skill directories for specific licenses. The `skill-creator` skill includes its own LICENSE.txt file.

## Author

Thurston Sandberg

## See Also

- [Claude Code Plugins Documentation](https://docs.claude.com/en/docs/claude-code/plugins)
- [Agent Skills Documentation](https://docs.claude.com/en/docs/claude-code/skills)
- [Plugin Marketplaces](https://docs.claude.com/en/docs/claude-code/plugin-marketplaces)
