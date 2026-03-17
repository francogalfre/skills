# skills

> Reusable AI agent skills for any coding assistant (opencode, Cursor, Claude Code, Windsurf, etc.)

<br />

## What's This

**Skills** are focused instruction sets that teach AI agents how to perform specific tasks effectively. Each skill contains detailed guidelines, patterns, and examples for common development workflows.

<br />

## Available Skills

| Skill | Description |
|-------|-------------|
| [system-prompt](system-prompt/) | Production-grade system prompts for AI SDKs (Vercel AI SDK, Anthropic, OpenAI), agents, and chatbots. |
| [api-docs](api-docs/) | Documents REST API endpoints with complete request/response examples, error codes, authentication, and parameters. |
| [conventional-commits](conventional-commits/) | Generates correct conventional commit messages from a diff, file list, or change description. |
| [pr-description](pr-description/) | Generates complete, structured pull request descriptions from diffs or commit lists. |
| [sst-aws](sst-aws/) | Build and deploy full-stack apps on AWS using SST v3 (Ion): stages, linking, secrets, and best practices. |

<br />

## Installation

Install any skill from this repository using the opencode CLI:

```bash
npx skills add francogalfre/skills --skill system-prompt
```

To list all available skills:

```bash
npx skills add francogalfre/skills --skill <name>
```

Replace `<name>` with any skill from the table above.

<br />

## Usage

Once installed, the skill automatically activates when you work on related tasks:

- **system-prompt**: Activates when writing, reviewing, or refactoring system prompts, designing agent instructions, building chatbots, or crafting prompts for generateText, streamText, generateObject, or tool calling.
- **api-docs**: Activates when the user says "document this endpoint", "write API docs", "generate API documentation", "add docs to my route", "write OpenAPI for this", or pastes a route handler and asks for documentation.
- **conventional-commits**: Activates when the user says "write a commit message", "generate a commit", "what should I name this commit", "help me commit this", or pastes a git diff or list of changed files.
- **pr-description**: Activates when the user says "write a PR description", "generate a pull request", "describe my changes", "write the PR for this diff", or pastes a git diff and asks for a description.
- **sst-aws**: Activates when working with `sst.config.ts`, SST components (Api, Function, Bucket, Queue, Dynamo, Postgres, Nextjs), `sst dev`, multi-stage deploys, secrets, or migrating SST v2 → v3.

<br />

## Contributing

Contributions are welcome! To add a new skill:

1. Create a new folder with a descriptive name
2. Add a `SKILL.md` file with detailed instructions
3. Update this README with the new skill

Open an issue or PR to suggest improvements.

<br />

## Local Development

To develop or test skills locally:

1. Copy the skill folder to your project's `.agents/skills/` directory
2. The agent will automatically pick up the skill instructions

```bash
mkdir -p .agents/skills
cp -r /path/to/skills/system-prompt .agents/skills/
```

<br />

<p align="center">
  <sub>Built for better AI-assisted development</sub>
</p>
