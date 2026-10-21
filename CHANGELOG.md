# Changelog

## v0.6 (2026-10-21)

Rules made precise where a review of the guide against a real code base found them ambiguous,
too strict for sound code, or too weak to stop a defect. Rules changed:

- Section 0: a waiver stands directly before the code it covers, with comments and tool
  directives allowed in between; it covers the next statement, declaration or macro call, and
  says so when it covers more.
- `FMT-13`: the project names its formatter, version and options; the formatter decides layout
  only; code where it is switched off follows the written rules; the project states whether
  indentation and line length are left to it. "Not changed by hand" is a MUST NOT.
- `NAM-9`: the `_n` suffix of active-low signals is a MUST.
- `CLS-8`: a constructor may also set the defaults of the class's own members.
- `LNG-20`: a cast of `null` succeeds; a result that must exist is also checked for `null`.
- `CMP-4`: a base class whose task phase starts stimulus says whether a derived class replaces
  or extends it; one that returns when its stimulus has ended may be extended with `super` first.
- `CMP-5`: a component whose job is the reset (a clock and reset driver, a reset monitor) keeps
  working during reset.
- `CMP-10`: the agent-level handle may be set in `build_phase` once the monitor has its port.
- `FAC-5`: applies to items, transactions and config objects; a service object implements the
  hooks it supports and says which.
- `CFG-4`: a mandatory value is checked with `uvm_fatal`; an optional lookup may go on without a
  value when it checks the handle and documents the fallback.
- `CFG-9`, `TLM-1`: status objects and services that the parent passes down are allowed;
  transaction streams always go through TLM.
- `TRN-6`: a consumer of an item with a status checks it before converting a 4-state value.
- `SEQ-2`: a sequence that sets fields after randomizing an item keeps the item's constraints,
  those of a factory override included, or says which it does not keep.
- `TLM-5`, `TLM-6`: a receiver changes no object that the received one refers to, and a clone
  copies what the receiver changes; non-class values are exempt.
- `MSG-1`: a project's own report macros that guard the call as the UVM macros do are allowed.
- `MSG-9`: the catcher matches the severity a message has when it reaches the catcher, and no
  earlier catcher may change it for that ID; a match after the check also fails the test.
- `TST-2`: the hooks are a MUST. `TST-7`: applies to tests of a DUT; a unit test of verification
  code checks its results directly.
- Appendix B: the factory line follows `FAC-1` and `FAC-2`, and checks the hooks of `FAC-5`.
- The guide and the README link to the reference testbench of hexk-sv-lib at tag `v0.11.0`.

## v0.5 (2026-10-17)

Rules changed so that code a formatter lays out, and reporting that a library builds on UVM,
conform as written:

- `FMT-13` (new): code that the project's formatter produces satisfies the layout rules that the
  formatter decides, and is not changed by hand; run the formatter on every change.
- `FMT-8`, `FMT-11`: where the formatter joins declarations or puts a space before `(`, its layout
  stands (`FMT-13`).
- `CLS-1`: a `get_type_name()` override of a parameterized class may come right before `new`.
- `CLS-8`: a constructor may also build helper objects that are part of the class and must exist
  before the build phase.
- `FAC-9`: a project's own report macros may call `uvm_report_*()` when they guard the call as the
  UVM macros do.
- `IFC-7`: non-class code that runs with UVM may report through UVM reporting.
- Appendix B checks `FMT-13`; Appendix C has a row for the formatter.
- The guide and the README link to the reference testbench of hexk-sv-lib at tag `v0.10.0`, which
  follows guide v0.4.

## v0.4 (2026-10-16)

Every rule now carries MUST, SHOULD or MAY. Section 0:

- Waivers: a deviation from a MUST or SHOULD is a `// <ID> waived: <reason>` comment at the code;
  the AI instructions say the same.
- Baseline: SystemVerilog IEEE 1800-2017; UVM 1.2 or IEEE 1800.2-2017/2020.
- Matching an existing file's style covers formatting and naming only. Generated and third-party
  code are exempt.
- A table of contents.

Keyword added, meaning unchanged: `FMT-5`, `FMT-8`, `FMT-11`, `DOC-2`, `LNG-21`, `CMP-8`, `CMP-9`,
`CMP-10`, `CMP-13`, `FAC-3`, `FAC-5`, `FAC-6`, `CFG-1`, `CFG-2`, `CFG-3`, `CFG-7`, `CFG-8`, `CFG-9`,
`IFC-1`, `IFC-6`, `IFC-7`, `IFC-9`, `TRN-1`, `TRN-4`, `SEQ-2`, `SEQ-4`, `SEQ-5`, `SEQ-9`, `SEQ-12`,
`SEQ-13`, `SEQ-15`, `OBJ-1`, `OBJ-6`, `TLM-1`, `TLM-2`, `TLM-3`, `TLM-4`, `CHK-1`, `CHK-2`, `CHK-3`,
`MSG-3`, `MSG-4`, `MSG-7`, `MSG-8`, `COV-1`, `COV-2`, `COV-4`, `COV-5`, `COV-8`, `COV-9`, `REG-1`,
`REG-2`, `REG-3`, `REG-6`, `REG-7`, `TST-1`, `TST-8`, `REU-1`, `REU-2`, `REU-3`, `REU-4`.

Rules changed:

- `FMT-2`: ASCII applies to SystemVerilog source and file lists; documentation may use UTF-8.
- `FMT-9`: two consecutive blank lines are a MUST NOT.
- `NAM-1`, `NAM-4`: names a library defines keep their spelling; an override keeps the argument
  names of the method it overrides. §2.3 lists more short forms; protocol acronyms are names.
- `DOC-5`: MUST for the public API of reusable code, SHOULD elsewhere.
- `DOC-8`: the unit of a quantity is a MUST.
- `LNG-15`: `local::` for every variable of the calling scope in an inline constraint.
- `LNG-16`: clear constraints first; staged or element-wise solves keep every constraint that spans
  them; a mask replaces `%` only where both give the same result.
- `FAC-1`: analysis FIFOs, exports and imps may be created with `new`.
- `FAC-2`: abstract UVM classes: the abstract registration macros under IEEE 1800.2, not
  registered under UVM 1.2.
- `FAC-10`: SHOULD, with the reason for type-based overrides.
- `IFC-4`: applies where the interface has clocking blocks and modports.
- `TRN-3`: metadata is never `rand` and is excluded from compare.
- `TRN-6`: a component that returns sampled values returns a non-OK status for an unknown value;
  negative tests tell consumers not to expect dropped transfers.
- `SEQ-10`: locks, grabs and objections are released on kill and on reset.
- `SEQ-14`: MUST NOT, with its exception.
- `OBJ-5`: MAY, after checking that nothing relies on intermediate objection totals.
- `OBJ-7`: a global timeout is required.
- `TLM-6` (new): a publisher creates a new object for every `write()` and does not change it later.
- `CHK-7`: the ISR sequence owns the locks; one lock order; release on completion, kill and reset;
  the handler states its service latency.
- `MSG-9`: a negative test counts its expected errors with a catcher on the reporting component that
  matches ID and original severity; the report server's per-ID count only under stated conditions.
  New catcher example.
- `COV-3`: accepted requests and completed responses at the DUT boundary are coverage; generated
  stimulus is not.
- `COV-7`: the reason is stated.
- `REG-4`: MUST, with the reason.
- `TST-6`: `$error`, SVA failures, simulator errors, abnormal ends and timeouts fail a run.

Examples:

- The interface example states its transfer contract; `request_stable_a` (was `valid_stable_a`)
  checks `write`, `addr`, and `wdata` for writes.
- The driver returns `UVM_HAS_X` for read data with an X or Z; `bus_item` says who checks
  `status`; the adapter checks for `null` before its cast.
- Appendix B is rebuilt with each rule's level and exceptions; Appendix C has rows for coverage
  goals and waivers.

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
