## How to Work With These Instructions

1. **Keep rules short and precise**  
   Copilot performs best when instructions are clean, atomic, and unambiguous. Long paragraphs or mixed-purpose rules reduce accuracy.

2. **Prefer strict rules over flexible guidelines**  
   The stricter and more explicit the rule, the higher the chance the model will correctly detect violations.  
   It is better when a rule is “triggered unexpectedly” once in a while than when an important rule is “not triggered at all”.

3. **Avoid “messy” or overly verbose prompts**  
   The longer the text and the more exceptions it contains, the higher the chance the model will hallucinate or misinterpret intent.

4. **Store additional context outside this ruleset**  
   Explanations, screenshots, historical notes, and extended documentation should live in Jira or Confluence.  
   → Add links to real Jira tickets or Confluence pages so reviewers can read the full context when needed.

5. **Do not duplicate generic programming best practices**  
   Copilot already checks standard quality issues (unused variables, missing awaits, etc.).  
   This ruleset should contain only **custom Playwright and project-specific conventions**, not generic programming patterns.



If you’d like to contribute improvements or propose additional rules, feel free to open an issue or pull request.
