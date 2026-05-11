Declaration:  declaration.md
Constitution: constitution.md
DAG:          docs/dag.md
State:        docs/state.md
Tests:        tests/

Execute the DAG from current state.
Dispatch independent tasks within each wave as parallel 
sub-agents where possible.
Run the test suite on completion of each wave.

On test failure: stop, report with full context, 
do not advance.
Update state on each step completion following the format defined in constitution.md.

When DAG is complete and all tests pass:
Open a PR referencing declaration, DAG completion 
summary, and wave-by-wave test results.
