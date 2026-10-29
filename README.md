# Architecture Skills

A collection of skills for software architecture design, review, and refactoring.

Each skill lives in its own folder under `skills/` and follows the `SKILL.md` format with YAML front matter.

## Design target

The included skill is optimized for **.NET** and **Vue/TypeScript** projects. Its code examples, naming conventions, and dependency rules are oriented toward that stack.

The architectural principles are broadly applicable and can be adapted to other languages, frameworks, and runtimes. When using another stack, translate the examples and adjust framework-specific details while preserving the dependency and boundary rules.

## Included skills

### `screaming-onion-architecture`

A skill that integrates four architecture ideas without treating them as synonyms:

- **Screaming Architecture** for organizing code by business capability.
- **Onion Architecture** for controlling dependency direction.
- **Clean Architecture** for defining boundaries and policies.
- **Vertical Slices** as an optional way to keep use-case flows cohesive.

The skill covers:

- MUST/SHOULD/MAY rules.
- Naming and folder conventions.
- A dependency matrix.
- A .NET authorization example.
- Gradual adoption of the architecture.
- Testing guidance.
- Reference standards.
- A Definition of Done.

## Structure

```text
architecture-skills/
├── README.md
└── skills/
    └── screaming-onion-architecture-skill/
        ├── SKILL.md
        └── README.md
```

## Usage

1. Copy the skill folder into the skills directory used by your agent or tool.
2. Make sure the agent loads `SKILL.md` as the skill instructions.
3. Enable it when designing, implementing, reviewing, or refactoring architecture.

## Scope

The architectural principles are domain-agnostic. The examples and implementation details target .NET and Vue/TypeScript, so adapt those parts when working with another stack.
