# Changelog

## v0.3 (2026-10-16)

Correctness fixes. Rules changed:

- `FIL-7`: every package, module and interface with a delay or a time value declares `timeunit`
  and `timeprecision`, before any `import`; a delay is rounded to the precision of its own scope.
  The package skeleton shows it.
- `CLS-8`: a constructor may build child config objects; it never creates child components.
- `LNG-20`: a cast whose failure is a bug is checked as a function and fails with `uvm_fatal`; the
  task form does not stop simulation.
- `LNG-21`: type dispatch handles a `null` source and the case where nothing matches.
- `CMP-4`: `super` first in every function phase; in task phases as the parent's header says. A base
  class whose task phase never returns or starts stimulus says so.
- `CMP-5`: what drivers, monitors and scoreboards do during reset.
- `CFG-3`: child config handles are `rand`.
- `CFG-10`: the order of the cfg hooks; pin a `rand` knob with `rand_mode(0)` before randomization;
  new hook `modify_randomized_cfg()` for plain assignments after it.
- `TRN-6` (new): monitors check valid fields for X/Z before storing them in 2-state fields.
- `SEQ-1`: stimulus lives in sequences even when only one test uses it; tests configure, select and
  start sequences.
- `MSG-9`: a negative test fails when the expected error does not occur the expected number of
  times, and on any other error.
- `REG-5`: when `provides_responses` is 0 and when it is 1.
- `TST-2`: new hooks `modify_randomized_cfg()` and `create_stimulus()`.

Examples:

- Appendix A: the driver and monitor handle reset; the monitor checks for X/Z; every request
  starts at a clock edge, so the driver no longer counts a handshake one cycle before the DUT
  could see it; new `bus_reg_adapter` and `bus_base_test` excerpt; the smoke test overrides
  `create_stimulus()` instead of `main_phase`.
- Appendix A is labelled as excerpts and says what they leave out; the guide and the README point
  to the reference testbench of hexk-sv-lib as a complete testbench that runs.
- The interface example samples `rst_n` in its clocking blocks and asserts that `valid` is known.
- The casting and cfg examples show the new rules.
- Appendix B checks the new rules; Appendix C states the `super` decision as `CMP-4` now does.

## v0.2 (2026-10-16)

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
