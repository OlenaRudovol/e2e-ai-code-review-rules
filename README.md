## Main principles used when wrining the Instructions

1. **Keep rules short and precise, prefer strict rules over flexible guidelines**  
   Copilot performs best when instructions are clean, atomic, and unambiguous. Long paragraphs or mixed-purpose rules reduce accuracy. 
   → It is better when a rule is “triggered unexpectedly” than when an important rule is “not triggered at all”.

2. **Store additional context outside this ruleset**  
   Explanations, screenshots, historical notes, and extended documentation should live in Jira or Confluence.  
   → Add links to real Jira tickets or Confluence pages so reviewers can read the full context when needed.

3. **Do not duplicate generic programming best practices**  
   Copilot already checks standard quality issues (unused variables, missing awaits, etc.).  
   This ruleset should contain only **custom Playwright and project-specific conventions**, not generic programming patterns.
