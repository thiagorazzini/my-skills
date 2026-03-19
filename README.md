# my-skills

A collection of custom skills for [Claude Code](https://claude.ai/code) — reusable, domain-specific prompts that extend Claude's behavior for specific tasks.

---

## What is a skill?

A skill is a markdown file (`SKILL.md`) that acts as a specialized prompt loaded into Claude when a specific task is needed. Instead of repeating long instructions every time, skills let Claude automatically know how to handle complex, multi-step workflows.

Each skill lives in its own folder and can include:
- `SKILL.md` — the main prompt that defines the skill's behavior
- `references/` — supporting docs, templates, and snippets the skill can read during execution
- `README.md` — documentation explaining what the skill does and how to trigger it

---

## How to use

### 1. Point Claude Code to this repository

In your project's `CLAUDE.md` (or globally in `~/.claude/CLAUDE.md`), add the path to the skills folder so Claude knows where to find them:

```md
## Skills
Load skills from: ~/skills
```

Or configure via Claude Code settings to auto-load skills from this directory.

### 2. Trigger a skill

Skills are triggered by natural language. Just describe what you want to do — Claude will recognize the intent and load the appropriate skill automatically.

Each skill's `README.md` documents which phrases trigger it.

---

## Available skills

| Skill | Description | Triggers |
|---|---|---|
| [dotnet-scaffold](dotnet-scaffold/) | Scaffolds a production-ready .NET Web API with Docker infrastructure, tests, and operational scripts | "create a new .NET project", "scaffold a C# API", "set up the backend" |

---

## Adding a new skill

1. Create a folder: `my-skill-name/`
2. Add `SKILL.md` with the prompt and instructions
3. Optionally add `references/` for supporting files
4. Add a `README.md` documenting what it does and how to trigger it
5. Register it in the table above
