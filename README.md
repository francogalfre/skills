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
| [accessibility-review](accesibility-review/) | Reviews UI components and HTML for accessibility issues based on WCAG 2.1 AA standards. |
| [bitbucket-pipelines](bitbucket-pipelines/) | Writes and debugs Bitbucket Pipelines CI/CD configuration. |
| [clean-code](clean-code/) | Refactors code to be cleaner, more readable, and easier to maintain — without changing behavior. |
| [code-review](code-review/) | Reviews code for bugs, security vulnerabilities, performance issues, and readability problems. |
| [debug-performance](debug-performance/) | Diagnoses and fixes performance problems in web apps: slow renders, memory leaks, large bundles, and Core Web Vitals. |
| [dependency-audit](dependency-audit/) | Audits package.json dependencies for outdated packages, security vulnerabilities, and unnecessary deps. |

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
- **accessibility-review**: Activates when the user says "check accessibility", "is this accessible", "WCAG review", "a11y audit", "fix accessibility issues", or pastes a React component, HTML, or JSX and asks for accessibility feedback.
- **bitbucket-pipelines**: Activates when the user says "set up Bitbucket Pipelines", "write a bitbucket-pipelines.yml", "my pipeline is failing", "add a deployment step", "configure CI for Bitbucket", or shares a bitbucket-pipelines.yml and asks for help.
- **clean-code**: Activates when the user says "clean up this code", "refactor this", "this is messy", "improve readability", "simplify this function", "too many nested ifs", or pastes code that works but is hard to read.
- **code-review**: Activates when the user says "review this code", "check my code", "what's wrong with this", "code review", "audit this file", or pastes code and asks for feedback.
- **debug-performance**: Activates when the user says "my app is slow", "too many re-renders", "memory leak", "LCP is bad", "bundle is too large", "this feels laggy", "optimize this component", or shares performance profiler output or Lighthouse results.
- **dependency-audit**: Activates when the user says "audit my dependencies", "check my packages", "what deps should I update", "clean up package.json", "are there security issues in my deps", or pastes a package.json.

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
