# Design Reference

Rease's UI is designed in Figma. Codex cannot log into Figma directly, so:

- Figma file: [PASTE YOUR FIGMA LINK HERE]
- For any task that builds a screen, the developer (not Codex) will paste
  the relevant frame as a screenshot/description into the prompt, or
  describe exact spacing, colors, and component states from Figma before
  asking for code.
- The design system tokens (color palette, typography scale, spacing
  units, dark/light theme values, the PIN-pad and bottom-sheet components)
  are documented in Figma under the "Design System" page — these should
  be transcribed into the app's theme file as an early Milestone 0 task,
  not re-derived by Codex from guesswork.
- When in doubt about a visual detail, Codex should ask the developer
  rather than inventing spacing, colors, or copy.