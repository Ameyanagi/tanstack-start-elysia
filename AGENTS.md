# Agent Guidance

This repository contains an Agent Skill for initializing full-stack monorepos.

## Repository Structure

- `skills/init-tanstack-elysia/SKILL.md` - Main skill instructions
- `skills/init-tanstack-elysia/references/stack-details.md` - All configuration file contents

## When modifying this skill

- Keep SKILL.md under 500 lines (currently ~200)
- All file contents belong in `references/stack-details.md`, not in SKILL.md
- The skill `name` in frontmatter must match the directory name
- Test changes by running `/init-tanstack-elysia` in a clean directory
- When updating dependency versions, update both the package.json templates and the Dockerfile base images
