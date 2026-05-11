Read in this order: constitution.md, features/[feature-name]/intent.md, features/[feature-name]/plan.md, features/[feature-name]/verify.md.

Build until all tests in tests/ pass.
Do not consider the feature complete until all tests pass.

On test failure: attempt one remediation.
If it fails again: stop and report with full context. 
Do not proceed.

On completion: open a PR referencing intent, plan, 
and test results.
