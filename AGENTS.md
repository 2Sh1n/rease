# Instructions for Codex

## Project context
Before doing anything, read these three files in order:
1. docs/PRODUCT_OVERVIEW.md — the full product spec, technical
   architecture, UI/UX documentation, database schema, and API contract.
2. docs/ROADMAP.md — the milestone-by-milestone task backlog. Every
   task I ask about maps to an entry in this file.
3. docs/DESIGN.md — where the Figma designs live and how to handle
   visual/UI questions.

## How to respond — always
- Never edit or create files directly. I am learning this codebase
  task by task and want to type the code in myself.
- For every task, respond with:
  1. A short explanation of what we're building and why.
  2. The exact file path(s) to create or edit.
  3. The full code for each file, in a fenced code block, ready to
     copy-paste.
  4. Any terminal commands to run, in their own code block.
  5. How to verify the task worked — what to check, what output
     to expect.
- If a change touches multiple files, list them in the order to
  create/edit them.
- Keep explanations beginner-friendly. I can read code, but this is
  my first time building a full-stack, multi-platform product from
  scratch.
- If a UI detail isn't specified in the prompt, ask rather than guess
  — check docs/DESIGN.md for where Figma specs would normally answer it.
- Follow the coding standards and git workflow in docs/PRODUCT_OVERVIEW.md
  §12 (TypeScript strict mode, Conventional Commits, squash merges,
  main/dev branch structure).