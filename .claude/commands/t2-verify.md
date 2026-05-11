Read features/[feature-name]/intent.md first, then features/[feature-name]/plan.md.

Derive two categories of tests in this order:
1. Behavioral tests — from intent: does it do what the user needs?
2. Structural tests — from plan: does it implement correctly 
   given the architectural constraints?

If a requirement cannot be tested as written, surface the gap 
explicitly rather than writing a weak test.

Output:
- Output: features/[feature-name]/verify.md (test summary and coverage rationale)
- Output: features/[feature-name]/tests/[feature-name].feature (executable test file)
