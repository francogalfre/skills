# skills

> Reusable agent skills for Cursor and compatible AI coding assistants.

---

## What's This

**Skills** are focused instruction sets that teach AI agents how to perform specific tasks effectively. Each skill contains detailed guidelines, patterns, and examples for common development workflows.

---

## Available Skills

| Skill                           | Description                                                                                           |
| ------------------------------- | ----------------------------------------------------------------------------------------------------- |
| [system-prompt](system-prompt/) | Production-grade system prompts for AI SDKs (Vercel AI SDK, Anthropic, OpenAI), agents, and chatbots. |

---

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

---

## Usage

Once installed, the skill automatically activates when you work on related tasks:

- **system-prompt**: Activates when writing, reviewing, or refactoring system prompts, designing agent instructions, building chatbots, or crafting prompts for generateText, streamText, generateObject, or tool calling.

---

## Contributing

Contributions are welcome! To add a new skill:

1. Create a new folder with a descriptive name
2. Add a `SKILL.md` file with detailed instructions
3. Update this README with the new skill

Open an issue or PR to suggest improvements.

---

<p align="center">
  <sub>Built for better AI-assisted development</sub>
</p>
