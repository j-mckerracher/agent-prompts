## Critical Rule: Follow Existing Patterns  
  
**All new files MUST follow the existing naming conventions in their directory.**  
For greenfield projects with no existing patterns, define the initial naming convention in the scaffolding UoW and apply it consistently.  
  
Before creating any new file:  
1. **Examine sibling files** in the same directory  
2. **Identify the naming pattern** used by existing files  
3. **Match that pattern exactly** for your new file  
  
### Example - What NOT to Do  
  
❌ **Wrong**: Creating `spec-acc-footer.test-harness.cy.ts` when no other files in the directory use "test-harness" in their names.  
  
```  
/libs/pearls/specimen-accessioning/ui/orders-ui/src/lib/components/spec-acc-footer/  
├── spec-acc-footer.component.ts  
├── spec-acc-footer.component.spec.ts  
├── spec-acc-footer.component.html  
└── spec-acc-footer.test-harness.cy.ts   ← WRONG: "test-harness" pattern doesn't exist  
```  
  
✅ **Correct**: Follow the existing `.component.` and `.spec.` pattern:  
  
```  
/libs/pearls/specimen-accessioning/ui/orders-ui/src/lib/components/spec-acc-footer/  
├── spec-acc-footer.component.ts  
├── spec-acc-footer.component.spec.ts  
├── spec-acc-footer.component.html  
└── spec-acc-footer.component.cy.ts      ← Matches existing pattern  
```  
  
### How to Check Existing Patterns  
  
Before naming a new file, run:  
```bash  
ls -la <directory>```  
  
Look for patterns like:  
- `*.component.ts` / `*.component.spec.ts`  
- `*.service.ts` / `*.service.spec.ts`  
- `*.cy.ts` for Cypress tests  
- `*.test.ts` vs `*.spec.ts` (use whichever exists)  
  
**When in doubt, match what's already there.**  
  
---  
  
## General Rules  
  
- Names MUST NOT contain any spaces. Use `-` (hyphen) in place of spaces.  
- Use lowercase with hyphens for top-level docs where appropriate.  
- **Never invent new naming patterns** when existing patterns are present.  
  
## Repo Structure  
  
- Prefer consistent naming with existing folders (`backend/`, `mobile/`, `progress-tracking/`).  
- Keep feature folders grouped by feature name (e.g., `profiles/`, `chat/`).  
- Match the casing and separator style of sibling directories.
