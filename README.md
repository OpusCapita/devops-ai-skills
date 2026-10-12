# devops-ai-skills

This repository stores OpenCode agent skills for DevOps work and personal knowledge areas.

## Structure

OpenCode discovers project-local skills from `.opencode/skills/<name>/SKILL.md`.
That means every skill lives under `.opencode/skills`, with one folder per skill.

More info about skills: https://opencode.ai/docs/skills/

## Main skills

These are the shared, team-facing skills in this repository. Each skill captures reusable knowledge, procedures and lessons learned.

- `shared-development`
Skills related to application development, including implementation patterns, architecture, coding standards, best practices, and guidelines for extending or modifying existing functionality.
- `shared-migration`
Skills covering application and infrastructure migrations, including migration strategies, step-by-step procedures, compatibility considerations, etc.
- `shared-issues`
Skills documenting problems that have occurred or may occur during development and maintenance. They describe symptoms, root causes, diagnostic approaches and proven solutions to help the team identify and resolve recurring issues efficiently.
- `shared-outages`
Skills derived from resolved outages and service disruptions. They capture incident details, root causes, recovery procedures, and preventive measures, turning operational experience into reusable knowledge to reduce the risk and impact of similar incidents in the future.

## Personal skills

These are the personal/private skills in this repository:

- `personal-damian`
- `personal-jonas`
- `personal-lukasz`
- `personal-michal`
- `personal-simon`

## Case and grouping

OpenCode treats skill names as lowercase identifiers, so capitalization is not a valid way to separate main and personal skills.

This repository uses name prefixes to make the scope explicit:

- `shared-*` for team-facing skills
- `personal-*` for private, person-owned skills

If you want to keep a shared vs personal distinction, use documentation, skill descriptions, or metadata. In this repository, the distinction is semantic, not based on folder case.

## Goal

The goal of this structure is to keep all OpenCode skills in the expected discovery path while making the shared and personal collections easy to understand and maintain.
