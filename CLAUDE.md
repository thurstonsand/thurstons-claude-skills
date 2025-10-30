# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **plugin marketplace** for Claude Code that hosts multiple independent plugins. Each plugin contains Agent Skills, slash commands, and agents that extend Claude's capabilities with specialized workflows and domain expertise.

**Architecture pattern:**
- Marketplace → Plugins → Skills/Commands/Agents
- Each plugin is self-contained with its own configuration
- Skills are automatically discovered and triggered based on task context

**Current plugins:**
1. **project-management-plugin**: Git workflows, code review, system architecture
2. **meta-plugin**: Tools for creating and managing skills/plugins
3. **homelab-plugin**: Infrastructure management (TrueNAS, Docker operations)

## Common Development Commands

### Creating a New Skill

Initialize a new skill with template structure:
```bash
python /home/thurstonsand/Code/claude-skills/meta-plugin/skills/skill-creator/scripts/init_skill.py <skill-name> --path <plugin-path>/skills
```

Example:
```bash
python /home/thurstonsand/Code/claude-skills/meta-plugin/skills/skill-creator/scripts/init_skill.py my-new-skill --path project-management-plugin/skills
```

### Validating and Packaging Skills

Package a skill for distribution (includes validation):
```bash
python meta-plugin/skills/skill-creator/scripts/package_skill.py <path/to/skill-folder> [output-directory]
```

Example:
```bash
python meta-plugin/skills/skill-creator/scripts/package_skill.py project-management-plugin/skills/git-commit-helper ./dist
```

### Testing Skills Locally

After making changes to skills, restart Claude Code to reload:
```bash
# In Claude Code CLI
/restart
```

Or test by adding the marketplace locally:
```bash
/plugin marketplace add /home/thurstonsand/Code/claude-skills
/plugin install <plugin-name>@claude-skills-marketplace
```

## Code Architecture

### Marketplace Structure

```
claude-skills/                          # Marketplace root
├── .claude-plugin/
│   └── marketplace.json                # Registers all plugins
├── .claude/
│   └── settings.local.json             # Local permissions for development
├── <plugin-name>/                      # Each plugin is a top-level directory
│   ├── .claude-plugin/
│   │   └── plugin.json                 # Plugin metadata and versioning
│   ├── skills/                         # Agent Skills directory
│   │   └── <skill-name>/
│   │       ├── SKILL.md                # Required: skill definition
│   │       ├── scripts/                # Optional: executable code
│   │       ├── references/             # Optional: documentation loaded into context
│   │       └── assets/                 # Optional: templates/files for output
│   ├── commands/                       # Slash commands (e.g., /ggc)
│   │   └── command-name.md
│   └── agents/                         # Multi-turn conversational agents
│       └── agent-name.md
└── README.md                           # Marketplace documentation
```

### Plugin Discovery and Registration

1. **Marketplace registration**: `.claude-plugin/marketplace.json` lists all plugins with relative paths
2. **Plugin metadata**: Each plugin has `.claude-plugin/plugin.json` with name, version, description, keywords
3. **Skill discovery**: Claude automatically detects skills based on metadata and user task context

### Skill Architecture (Agent Skills)

Every skill follows this pattern:

**Required: SKILL.md**
```yaml
---
name: skill-identifier              # kebab-case, max 64 chars
description: What it does and WHEN to use it  # max 1024 chars, include trigger scenarios
allowed-tools: [optional]           # Restrict tool access if needed
---

# Skill Name

Markdown instructions for Claude (use imperative/infinitive form, not second person)
```

**Optional: Bundled Resources**

Three types of bundled resources, each with different context management:

1. **scripts/** - Executable code (Python/Bash)
   - Purpose: Deterministic operations, complex escaping, automation
   - Context: May be executed without loading into context, but can be read for patching
   - Examples: `init_skill.py`, `docker_exec_python.sh`, `create_pr.py`

2. **references/** - Documentation for context loading
   - Purpose: Large documentation, API specs, schemas, detailed guides
   - Context: Loaded into context only when Claude determines it's needed
   - Examples: `server_layout.md`, API references, workflow guides
   - Best practice: For files >10k words, include grep patterns in SKILL.md

3. **assets/** - Output resources
   - Purpose: Templates, boilerplate, files to copy/use in output
   - Context: Never loaded into context, used in final output
   - Examples: LICENSE.txt, document templates, boilerplate code

**Path conventions:**
- Always use forward slashes in SKILL.md references: `scripts/helper.py` (not `scripts\helper.py`)
- Paths are relative to the skill directory

### Naming Conventions

**Plugins:**
- Kebab-case with `-plugin` suffix
- Examples: `project-management-plugin`, `meta-plugin`, `homelab-plugin`

**Skills:**
- Kebab-case identifiers
- Descriptive of purpose
- Common patterns: `verb-noun` (git-commit-helper), `service-operation` (truenas-docker-ops), `compound-noun` (skill-creator)

**Commands:**
- Lowercase, often acronyms or short verbs
- Examples: `/ggc`, `/system-architect`

**Directories:**
- All lowercase
- Standard directories: `skills/`, `commands/`, `agents/`, `scripts/`, `references/`, `assets/`

### Progressive Disclosure Pattern

Skills use progressive disclosure to manage context efficiently:

1. **Metadata** (~100 words): Always in context (name, description from YAML frontmatter)
2. **SKILL.md** (~<5k words): Loaded when skill is triggered
3. **Bundled resources**: Loaded/executed only as needed

This allows the marketplace to scale to hundreds of skills without context bloat.

### Documentation Style

All skill documentation uses **imperative/infinitive form** (not second person):
- ✓ "To accomplish X, do Y"
- ✓ "Run the command..."
- ✗ "You should do X"
- ✗ "If you need to do X"

This maintains consistency for AI consumption and follows Claude Code conventions.

## Key Configuration Files

### `.claude-plugin/marketplace.json`
Registers the marketplace and all plugins. Uses relative paths to allow plugins to be self-contained:
```json
{
  "name": "claude-skills-marketplace",
  "owner": { "name": "Thurston Sandberg" },
  "plugins": [
    {
      "name": "plugin-name",
      "source": "./plugin-name",
      "description": "..."
    }
  ]
}
```

### `<plugin>/.claude-plugin/plugin.json`
Per-plugin configuration with versioning and discovery metadata:
```json
{
  "name": "plugin-name",
  "version": "1.0.0",
  "description": "...",
  "author": { "name": "..." },
  "keywords": ["tag1", "tag2"]
}
```

### `.claude/settings.local.json`
Local development permissions for running scripts and reading files without prompts:
```json
{
  "permissions": {
    "allow": [
      "Bash(python /path/to/scripts/*.py:*)",
      "Read(/path/to/files/**)"
    ],
    "deny": [],
    "ask": []
  }
}
```

## Working with Skills

### When to Create a New Skill vs. Modify Existing

**Create a new skill when:**
- Adding functionality for a new domain or use case
- The functionality is triggered by different user scenarios
- The skill would have a distinct description that helps Claude decide when to use it

**Modify existing skill when:**
- Enhancing or fixing existing functionality
- Adding examples or clarifying instructions
- The change fits within the existing skill's trigger scenarios

### Skill Best Practices

1. **Focused scope**: Each skill addresses one capability. Break broad functionality into separate skills.

2. **Clear descriptions**: Include both WHAT the skill does and WHEN Claude should use it. Include specific terms users would mention.

3. **Imperative instructions**: Use imperative/infinitive form in all documentation, not second person.

4. **Progressive disclosure**: Put core instructions in SKILL.md, detailed docs in references/, and use scripts for complex operations.

5. **Tool restrictions**: Use `allowed-tools` in YAML frontmatter for read-only or limited-scope skills.

6. **Validation before packaging**: Always run `package_skill.py` which validates structure, naming, and description quality.

### Bundled Resource Guidelines

**When to use scripts/:**
- Complex shell operations with escaping (e.g., nested SSH + docker exec)
- API interactions (e.g., GitHub PR creation)
- Data processing utilities
- Any deterministic operation that's better as executable code

**When to use references/:**
- API documentation
- Database schemas
- Workflow guides longer than ~2k words
- Domain-specific knowledge that informs Claude's process

**When to use assets/:**
- Document templates (PPTX, DOCX, etc.)
- Boilerplate code directories
- License files
- Any file meant to be copied/used in output, not loaded into context

## Testing and Validation

### Validation Process

The `package_skill.py` script validates:
- YAML frontmatter structure and required fields
- Naming conventions (kebab-case, max 64 chars)
- Description quality (must include trigger scenarios)
- SKILL.md file existence
- Directory structure

Run validation:
```bash
python meta-plugin/skills/skill-creator/scripts/package_skill.py <skill-path>
```

### Local Testing Workflow

1. Make changes to skill files
2. Restart Claude Code to reload skills: `/restart`
3. Test the skill by describing a scenario that should trigger it
4. Verify Claude uses the correct skill and follows instructions
5. If needed, adjust description or instructions and repeat

### Integration Testing

After creating or modifying skills:
1. Package the skill to validate structure
2. Test installation from local marketplace
3. Verify skill appears in plugin list: `/plugin list`
4. Test multiple trigger scenarios to ensure proper activation

## Contributing to the Marketplace

### Adding a New Plugin

1. Create plugin directory: `<plugin-name>/`
2. Add plugin configuration: `<plugin-name>/.claude-plugin/plugin.json`
3. Create skills, commands, or agents in appropriate subdirectories
4. Register plugin in `.claude-plugin/marketplace.json`
5. Test locally before committing

### Adding a New Skill to Existing Plugin

1. Initialize skill:
   ```bash
   python meta-plugin/skills/skill-creator/scripts/init_skill.py <skill-name> --path <plugin>/skills
   ```
2. Edit `SKILL.md` to replace all TODO items
3. Add scripts/references/assets as needed (delete unused directories)
4. Validate and test:
   ```bash
   python meta-plugin/skills/skill-creator/scripts/package_skill.py <plugin>/skills/<skill-name>
   ```
5. Test locally with `/restart` and trigger scenarios
6. Commit to repository

## Important Patterns

### Script Execution Pattern

Skills with scripts follow this pattern in SKILL.md:
```markdown
## Using the Helper Script

Execute the script to [accomplish task]:

```bash
python skills/<skill-name>/scripts/helper.py [args]
```
```

The script path is always relative to the repository root.

### Reference Documentation Pattern

For large documentation files, SKILL.md should:
1. Provide an overview of what's in the reference
2. Explain when Claude should load it
3. Optionally include grep patterns for specific sections

Example:
```markdown
## Reference Documentation

See `references/api_docs.md` for complete API specifications.

Key sections:
- Authentication: Line 15-45
- Endpoints: Line 50-200
- Error codes: Line 205-250
```

### Multi-Step Workflow Pattern

Skills that guide multi-step workflows should:
1. Start with a decision tree or quick start guide
2. Break steps into clear numbered sections
3. Include examples with realistic user requests
4. Reference scripts where deterministic operations are needed

## Documentation Requirements

When adding or modifying skills:
1. **SKILL.md**: Always update with clear, imperative instructions
2. **README.md**: Update marketplace README if adding new plugin or significantly changing functionality
3. **plugin.json**: Update version number following semantic versioning
4. **marketplace.json**: Update if adding/removing plugins

## Common Pitfalls

1. **Path separators**: Always use forward slashes `/` in skill references, never backslashes `\`
2. **Second person**: Avoid "you should" - use imperative form instead
3. **Monolithic skills**: Break broad functionality into focused, single-purpose skills
4. **Missing trigger scenarios**: Description must explain WHEN to use the skill, not just WHAT it does
5. **Context bloat**: Put large documentation in references/, not SKILL.md
6. **Windows paths**: Even on Windows, use forward slashes in SKILL.md references

## External Resources

- [Claude Code Skills Documentation](https://docs.claude.com/en/docs/claude-code/skills)
- [Claude Code Plugins Documentation](https://docs.claude.com/en/docs/claude-code/plugins)
- [Plugin Marketplaces](https://docs.claude.com/en/docs/claude-code/plugin-marketplaces)
