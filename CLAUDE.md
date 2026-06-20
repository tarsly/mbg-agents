# mbg-agents — Repo Guide

This is the **MBG Foundation** Claude Code plugin marketplace. It hosts the guided onboarding skills every MBG client runs first (`01 Clarify Your Goals`, `02 Map Your Business`, `03 Design Your AI Team`). Output from these skills (`goal-hierarchy`, `business blueprint`, `Agent Team`) is consumed by plugins in the sibling `mbg-library` repo.

Top-level layout:

- `.claude-plugin/marketplace.json` — the marketplace manifest. Every plugin must be listed here.
- `plugins/<plugin-name>/` — one directory per plugin.
- `plugins/<plugin-name>/.claude-plugin/plugin.json` — that plugin's own manifest.
- `plugins/<plugin-name>/skills/<skill-name>/SKILL.md` — one directory per skill, frontmatter + markdown body.

---

## Version Bumps — Required

**Whenever you modify an existing plugin's files (adding a skill, fixing a bug, updating a description), you MUST bump that plugin's version in two places:**

1. `plugins/<plugin-name>/.claude-plugin/plugin.json` → `"version"`
2. `.claude-plugin/marketplace.json` → the matching plugin entry → `"version"`

The two must stay in sync. Drift is invisible until someone installs the plugin and gets confused about which version they have.

**Semver rules:**
- **Patch** (`1.0.0` → `1.0.1`) — bug fixes, typo fixes, doc clarifications, error-handling improvements
- **Minor** (`1.0.0` → `1.1.0`) — new skill added, new capability on an existing skill, new optional argument
- **Major** (`1.0.0` → `2.0.0`) — breaking change: renamed/removed skill, changed argument semantics, changed Cloud Brain note title or folder

**New plugins start at `1.0.0`.** No bump needed on the initial commit that creates them.

---

## Plugin Naming

- Plugin directory name, `plugin.json` `name` field, and `marketplace.json` `name` field must all match.
- Kebab-case identifiers aligned with the `displayName` (see commit `6a6e33c Add displayName "Start Here" to plugin manifests`).

---

## Skill Naming — Ordinal Prefix

Skills in this repo are guided foundation skills meant to run **in sequence**. They use a numeric ordinal prefix instead of the functional prefix used in `mbg-library`:

- ✅ `01-clarify-your-goals`
- ✅ `02-map-your-business`
- ✅ `03-design-your-ai-team`
- ❌ `foundation-clarify-your-goals` (don't use functional prefix here)

When adding a new foundation skill, pick the next ordinal (`04-`, `05-`, …). If inserting between existing ones, renumber subsequent skills and bump the plugin's **minor** version.

---

## SKILL.md Structure

Every `SKILL.md` follows the same 9-section structure used across MBG plugins. Reference: `plugins/start-here/skills/01-clarify-your-goals/SKILL.md`.

1. YAML frontmatter — `name`, `description`, `argument-hint`, `allowed-tools`
2. Overview
3. When This Skill Applies
4. Pre-Flight — Preferences (search Cloud Brain; ask in ONE message if missing; render banner)
5. How It Works
6. Data Structure (markdown templates for any notes the skill writes)
7. Output Format
8. Example Usage
9. Error Handling

---

## Cloud Brain Storage

These foundation skills WRITE the notes that downstream plugins READ:

| Note title | Folder | Written by | Read by (examples) |
|------------|--------|------------|--------------------|
| `goal-hierarchy` | `goals` | `01-clarify-your-goals` | `mbg-admin/goals-pulse`, `eos-vto-builder` |
| `quarterly-priorities` | `goals` | `01-clarify-your-goals` | `mbg-admin/goals-pulse`, `eos-rocks` |
| `language-and-frameworks` | `preferences` | `01-clarify-your-goals` | every downstream skill |
| `business blueprint` | `business` | `02-map-your-business` | `agent-designer`, every EOS skill |
| `Agent Team` | `agents` | `03-design-your-ai-team` | `agent-activator`, `eos-accountability-chart` |

**Implication:** changing a note title, folder, or required field on ANY of these is a **breaking change** to downstream plugins in `mbg-library`. Bump the major version and announce.

Cloud Brain rules (same as `mbg-library`):

- The `folder` parameter on `write_note` must NEVER include a `brain/` prefix. Use `goals`, `business`, `agents`, `preferences`, etc. directly. (Cloud Brain writes `<folder>/<title>.md` directly.)
- Tags are YAML lists, not JSON-array strings: `tags=["a","b"]`, not `tags='["a","b"]'`.
- Preferences go under `brain/preferences/` (legacy convention preserved from the basic-memory migration).

---

## Pre-Commit Checks

Before committing changes to a plugin:

1. `jq . plugins/<plugin>/.claude-plugin/plugin.json` — must succeed.
2. `jq . .claude-plugin/marketplace.json` — must succeed.
3. Confirm the plugin's `version` matches in both manifests.
4. Confirm every new/modified `SKILL.md` has a frontmatter block with `name`, `description`, and `allowed-tools`.
5. If you changed a Cloud Brain note title/folder/schema, confirm you bumped the **major** version and noted the downstream impact in the commit message.

---

## Sibling Repo

The functional/operational plugins live in `~/repos/mbg-library` (separate marketplace, separate GitHub repo). Skills there assume the notes documented above already exist. Coordinate breaking changes across both repos.
