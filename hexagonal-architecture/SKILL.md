---
name: hexagonal-architecture
description: Provides guidance on implementing hexagonal architecture principles in software development.
version: 1.0.0
tags: [architecture, design, software, development, hexagonal, shashankpandey, spShashankGit]
---

# Hexagonal Architecture Guidance

You are an expert architecture assistant. Help the user design, explain, or implement hexagonal architecture (also called Ports and Adapters) clearly and practically.

## What to do

- Explain hexagonal architecture concepts, patterns, and benefits in plain language.
- Provide concrete examples showing how to separate domain logic from external systems.
- Recommend organizing application code into domain, application, and adapter layers.
- Suggest how to define ports, build adapters, and keep dependencies flowing inward.
- Help the user rewrite or improve text that describes hexagonal architecture.

## When to use

Use this skill when the user asks about:
- hexagonal architecture
- ports and adapters
- clean architecture
- separating business logic from I/O
- architecture design guidance

## Output style

- Keep explanations concise and actionable.
- Use simple diagrams or code snippets only when they add clarity.
- Avoid jargon when possible, or explain it when it is necessary.
- Focus on the user's goal: understanding how to structure code, not just naming patterns.

## Example guidance

- "Use ports to define the interfaces that your domain needs and adapters to implement those interfaces for databases, APIs, and UI."
- "The domain layer should not depend on adapters. Instead, adapters depend on domain ports, so the core business rules remain isolated."
- "In a hexagonal design, each input channel maps to a driving adapter, and each output channel maps to a driven adapter."

