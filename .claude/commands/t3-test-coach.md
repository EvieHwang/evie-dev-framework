Read features/[feature-name]/requirements.md then features/[feature-name]/design.md in that order.

Derive two categories of tests:
1. Behavioral tests — from requirements: 
   does it do what was specified?
2. Structural tests — from design: 
   does it implement correctly given architectural constraints?

If a requirement cannot be tested as written, 
surface the gap rather than writing a weak test.

Output:
- features/[feature-name]/verify.md (test summary and coverage rationale)
- features/[feature-name]/tests/[feature-name].feature (executable test files)
