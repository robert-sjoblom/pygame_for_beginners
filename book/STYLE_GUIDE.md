# Book Style Guide: Making Games with Python and Pygame

## Audience
- Kids aged 10-14 with minimal computer literacy
- Self-study format (no instructor present)
- Running Debian Linux

## Tone & Voice
- Straight but simple. Respect the reader's intelligence but don't use jargon without explaining it.
- Use "you" to address the reader. Use "we" when working through something together.
- Use "they/them" for hypothetical people. Never assume gender.
- Be opinionated: say "do it this way" rather than offering multiple options.
- Acknowledge when something is hard: "This part is tricky. Read it twice if you need to."
- Non-violent content. Games can have enemies and obstacles but keep it cartoon-style.
- No emojis in the text.
- Weave in programming culture naturally (mention real games, real programmers, open source).

## Chapter Structure
Each chapter follows this template:

```
# Chapter N: Title

## What You Will Learn
- Bullet point 1
- Bullet point 2
- Bullet point 3

## Main content sections...
(2-3 major sections with ### sub-sections)
(Clear stopping points marked with --- between major sections)

## What You Learned
Brief recap of key concepts.

## Quick Quiz
3-4 short questions testing understanding.

## Experiment
2-3 "try changing X and see what happens" prompts.

## Chapter Challenge
One coding challenge that uses everything from the chapter.
Provide progressive hints.
```

## Code Style
- Build-up approach: start with empty file, add lines, explain, add more
- Python snake_case with simple English words: player_x, enemy_speed, score
- Keep individual code blocks under 15 lines where possible
- Every code block should be something the reader types out (encourage typing, not copy-paste)
- Include checkpoints: "Your code should now look like this:" with full listing
- Comments in code should be minimal and only where truly helpful
- Always show expected output after code that produces output

## ASCII Diagrams
Use ASCII art to illustrate:
- Coordinate systems
- Game loops
- Data structures (lists, dictionaries)
- Game state transitions
- Screen layouts

## Math
Explain any math concept when it first appears. Don't assume they know coordinates, angles, or anything beyond basic arithmetic.

## Problem Solving
Model thinking before coding: "Let's think about what we need first..."
Introduce pseudocode early.

## Error Handling (Progressive)
- Early chapters: careful instructions to prevent errors
- Middle chapters: start showing common error messages and what they mean
- Late chapters: expect the reader to debug with guidance

## Session Length
Target 15-20 minute reading/coding sessions. Mark natural stopping points with horizontal rules (---).

## Formatting Conventions
- **Bold** for new terms when first introduced
- `backticks` for code, file names, commands, and keyboard keys
- Code blocks with python syntax highlighting
- > Blockquotes for "Did you know?" sidebars and tips
- ASCII diagrams in code blocks (no syntax highlighting)

## File naming
- Chapters: chapter_01.md through chapter_15.md
- Use zero-padded numbers for sorting
