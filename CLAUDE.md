# spaceship-skills

Agent skills for managing domains via the Spaceship registrar API. Companion to the [spaceship-mcp](https://github.com/bartwaardenburg/spaceship-mcp) server.

## Repository Structure

```
.claude-plugin/
  marketplace.json          # Marketplace metadata
  plugin.json               # Plugin metadata
spaceship-domains/          # Domain management skill
  SKILL.md                  # Skill entry point
  references/
    api-reference.md        # Complete API parameter specs
    gotchas.md              # Common pitfalls and edge cases
    patterns.md             # DNS recipes and deployment patterns
.github/
  workflows/
    validate.yml            # CI validation
```

## Creating a New Skill

1. Create a directory with a kebab-case name (e.g. `spaceship-hosting/`)
2. Add a `SKILL.md` with YAML frontmatter:
   - `name` (required): kebab-case skill name
   - `description` (required): Third-person, includes trigger phrases, 1-2 sentences
3. Add a `references/` directory for supplementary documentation
4. Update `.claude-plugin/plugin.json` if adding to the plugin

### SKILL.md Requirements

- Under 500 lines — keep focused, put details in references
- Include "When to Use" and "When NOT to Use" sections
- Include step-by-step workflows for common tasks
- Use progressive disclosure: SKILL.md links to reference files, references don't chain
- No hardcoded file paths

### Reference Files

- `api-reference.md` — parameter specs and return types
- `gotchas.md` — common pitfalls with examples
- `patterns.md` — recipes and common configurations

### Naming Conventions

- Directories: kebab-case (`spaceship-domains`)
- Skill names: kebab-case (`spaceship-domains`)
- Reference files: kebab-case (`api-reference.md`)

## Quality Standards

- Description must include trigger phrases ("Use when asked to...")
- Every gotcha must include correct vs incorrect example
- Workflows must be numbered step-by-step
- Financial/destructive operations must note user confirmation requirement
- Test all tool calls against the actual MCP server before documenting

## PR Checklist

- [ ] SKILL.md has valid YAML frontmatter with `name` and `description`
- [ ] No hardcoded file paths
- [ ] Skill is under 500 lines
- [ ] "When to Use" and "When NOT to Use" sections present
- [ ] All JSON files are valid
- [ ] marketplace.json and plugin.json versions are in sync
