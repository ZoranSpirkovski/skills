# Skills Marketplace

This repo is a Claude Code plugin marketplace. Each plugin in `plugins/` contains one or more skills.

## Adding a New Plugin

### 1. Create the directory structure

```
plugins/<plugin-name>/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── <skill-name>/
        ├── SKILL.md
        └── (optional supporting files like references/, templates/, etc.)
```

### 2. Write the SKILL.md

Every skill needs YAML frontmatter with `name` and `description`:

```markdown
---
name: my-skill
description: Use when [trigger context] — [what it does]
---

# Skill content here
```

The `description` tells Claude when to invoke the skill. Make it specific about trigger conditions.

### 3. Create plugin.json

Minimal manifest — skills in `skills/` are auto-discovered:

```json
{
  "name": "<plugin-name>",
  "description": "Brief description of what this plugin does",
  "version": "1.0.0"
}
```

Do NOT list individual skill files in a `skills` field. The `skills/` directory is auto-discovered.

### 4. Register in marketplace.json

Add an entry to `.claude-plugin/marketplace.json` in the `plugins` array:

```json
{
  "name": "<plugin-name>",
  "description": "Same or similar description",
  "source": "./plugins/<plugin-name>",
  "version": "1.0.0"
}
```

### 5. Update README.md

Add the plugin to the "Available Plugins" table and create a section under "Plugins" with description and install command:

```
/plugin install <plugin-name>@skills
```

### 6. Validate

Run `claude plugin validate .` from the repo root and `claude plugin validate plugins/<plugin-name>` for the new plugin. Both must pass.

## Updating an Existing Plugin

- Edit files in `plugins/<plugin-name>/skills/` directly.
- Bump `version` in plugin.json when making changes users should receive.
- Update the marketplace.json version if the version is managed there instead.

## Rules

- Plugin names use kebab-case.
- One plugin.json per plugin, inside `.claude-plugin/`.
- Skills go in `skills/<skill-name>/SKILL.md`, not in `.claude-plugin/`.
- Supporting/reference files go alongside SKILL.md (e.g., `skills/<skill-name>/references/`).
- All paths in plugin.json must start with `./` if specified.
- Don't duplicate version in both plugin.json and marketplace.json — pick one location.
