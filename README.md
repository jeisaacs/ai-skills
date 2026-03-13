# ai-skills

![Skills](https://img.shields.io/badge/skills-6-blue)
![Platform](https://img.shields.io/badge/platform-Claude%20Code%20%7C%20ChatGPT-purple)
![License](https://img.shields.io/badge/license-MIT-green)
![Contributions](https://img.shields.io/badge/contributions-welcome-orange)

A collection of AI coding assistant skills for Claude Code and ChatGPT custom instructions. Each skill teaches the AI a specific workflow so it can handle structured tasks autonomously.

---

## Skills

| Skill | Description |
|-------|-------------|
| [terraform-wizard](./terraform-wizard) | Comprehensive Terraform/OpenTofu guidance — modules, testing, CI/CD, security scanning, and production patterns. |
| [code-review-response](./code-review-response) | Processes open PR review comments (human or bot), triages suggestions, applies one-commit-per-comment fixes, runs tests, and resolves threads. |
| [commit-text](./commit-text) | Generates a paste-ready conventional commit message from your staged/unstaged changes and copies it to clipboard. No AI attribution included. |
| [linear-ticket-update](./linear-ticket-update) | Syncs a Linear ticket with the actual implementation — updates title, description, technical details, and acceptance criteria from the branch diff. |
| [documentation-enforcer](./documentation-enforcer) | Reviews source files for documentation quality, generates unified diffs with suggested docstrings/comments, and produces a summary report without modifying files. |
| [uncodixfy](./uncodixfy) | Prevents generic AI/Codex UI patterns in frontend code. Enforces clean, human-designed aesthetics inspired by Linear, Raycast, Stripe, and GitHub. |

---

## Installation

### Claude Code

1. Clone this repo (or copy individual skill folders):
   ```bash
   git clone https://github.com/jeisaacs/ai-skills.git
   ```

2. Add skills to your project's `.claude/skills/` directory:
   ```bash
   # Copy a single skill
   cp -r ai-skills/terraform-wizard /path/to/your-project/.claude/skills/

   # Or symlink all skills
   ln -s /path/to/ai-skills/* /path/to/your-project/.claude/skills/
   ```

3. Claude Code will automatically detect and load skills from `.claude/skills/`. Each skill has a `SKILL.md` (or `skill.md`) that defines when and how it activates.

### ChatGPT (Custom Instructions / GPTs)

1. Open the `SKILL.md` file for the skill you want.
2. Copy the full contents.
3. Paste into one of:
   - **Custom Instructions** → "What would you like ChatGPT to know?" or "How would you like ChatGPT to respond?"
   - **GPT Builder** → Instructions field when creating a custom GPT
   - **Project Instructions** → In a ChatGPT Project's instructions panel

> **Note:** ChatGPT has a character limit on instructions. For large skills like `terraform-wizard`, you may need to trim the reference sections or split across system/user prompts.

---

## Attribution

| Skill | Author | Source |
|-------|--------|--------|
| terraform-wizard | [Anton Babenko](https://github.com/antonbabenko) | [terraform-skill](https://github.com/antonbabenko/terraform-skill) (Apache-2.0) |
| uncodixfy | [cyxzdev](https://github.com/cyxzdev) | [Uncodixfy](https://github.com/cyxzdev/Uncodixfy) |
| All other skills | [Jonathan Isaacs](https://github.com/jeisaacs) | This repo |

---

## Contributing

Have a skill to share? PRs are welcome. Each skill should live in its own folder with a `SKILL.md` file containing YAML frontmatter (`name`, `description`) and the full skill prompt.
