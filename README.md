## Main principles used when wrining the Instructions

1. **Keep rules short and precise, prefer strict rules over flexible guidelines**  
   Copilot performs best when instructions are clean, atomic, and unambiguous. Long paragraphs or mixed-purpose rules reduce accuracy. 
   → It is better when a rule is “triggered unexpectedly” than when an important rule is “not triggered at all”.

2. **Store additional context outside this ruleset**  
   Extended documentation, screenshots, tables and diagrams and other information should live in externally (Jira, Confluence or any website).  
   → Add links to these resources so reviewers can follow them and read the full context.

3. **Do not duplicate generic programming best practices**  
   Copilot already checks standard quality issues (unused variables, missing awaits, etc.).  
   → The ruleset should contain only **custom project-specific conventions**, not generic programming patterns.
