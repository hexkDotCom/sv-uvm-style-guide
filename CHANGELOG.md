# Changelog

## Unreleased

Rules narrowed where they could not apply as written:

- `NAM-4`: where a linter treats an argument that shadows a member as an error, the argument
  takes a descriptive name; a prefix or suffix that only marks an argument stays forbidden.
- `FIL-3`: a self-contained example or unit-test file may declare its classes in its module.
- `CLS-8`, `FAC-2`, `MSG-1`: apply to UVM classes; plain utility classes, interface classes and
  classes of a UVM-free package are exempt, and the latter report with the severity tasks.
- `LNG-10`: a unit test of a randomizer may check an expected randomization failure.
- `CMP-14`: the ban on consuming time is about protocol timing; supervision and timing
  utilities may wait.
- `CFG-5`: the `REG::` resources of UVM's register sequences go through `uvm_resource_db`.
- `COV-7`: weight 0 is allowed on coverpoints that only build a cross.
- `TST-4`: a top may pass `+UVM_TESTNAME` to `run_test()` itself to run without UVM's
  command-line DPI; a self-contained example may pass its own test name.

## v0.1 (2026-10-09)

First public draft.
