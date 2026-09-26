# Skill Routing

Read this file first, then read the listed skills for your role.

If a listed skill file is missing, stop and ask the user. Do not invent content.

| Role | Required Skills | Reason |
|------|-----------------|--------|
| lead | `skills/scripting.md`, `skills/api-lookup.md` | Understand structure and API scope before delegating |
| writer | `skills/basics.md`, `skills/scripting.md`, `skills/performance.md`, `skills/api-lookup.md`, `skills/networking.md` | Full build context: syntax, runtime, speed, API, remotes |
| checker | `rules.md`, `skills/scripting.md`, `skills/performance.md` | Verify structure, error handling, performance |

## Usage

1. Identify your role.
2. Read each file in order.
3. Apply rules from `rules.md` (loaded globally via config).
4. Handoff: basics -> scripting -> performance -> api-lookup -> networking.
