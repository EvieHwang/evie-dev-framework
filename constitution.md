# Constitution

## Standards
Interface:        Apple Human Interface Guidelines
                  developer.apple.com/design/human-interface-guidelines
Accessibility:    WCAG 2.1 AA
                  w3.org/WAI/WCAG21/quickref
Security:         OWASP Top 10
                  owasp.org/www-project-top-ten
API contracts:    OpenAPI Specification
                  spec.openapis.org

Extended (apply when relevant):
API design:       Microsoft REST API Guidelines
AI integration:   OWASP Top 10 for LLMs
Security depth:   OWASP ASVS

Agents follow these without deviation unless the declaration 
explicitly requires otherwise. Deviation must be surfaced as 
a decision, not made silently.

## Architectural principles
[rules that govern how components relate — populated as decided]

## Patterns in use
[established conventions — file structure, naming, error handling]

## Quality gates
[what must be true before any PR — unconditional]

## Out of scope
[what this codebase explicitly does not do]

## Decision log
| Date | Decision | Rationale |
|------|----------|-----------|
[populated as significant decisions are made]

## Artifact formats

### State file
Location: features/[feature-name]-[number]/state.md
Format:
| ID | Description | Wave | Status | Notes |
|----|-------------|------|--------|-------|

Valid status values: pending, in-progress, complete, failed
Updated by: t3-generate-dag (initializes), t3-next-step (updates)
