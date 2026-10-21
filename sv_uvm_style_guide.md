# HEXK SystemVerilog / UVM Style Guide

Version 0.6 · 2026-10-21 · Scope: verification code (testbench, UVM, VIP). RTL is out of scope.

Copyright 2026 HEXK. Text: [CC BY 4.0](LICENSE-docs). Code examples: [Apache-2.0](LICENSE).

This guide is written for two readers: verification engineers and AI coding assistants.
Every rule has a stable ID (for example `NAM-4`) so that reviews, lint waivers and AI
prompts can cite it. Rules are short; the reason follows in *italics* only when it is not obvious.

**Contents**

- [0. How to read and apply this guide](#0-how-to-read-and-apply-this-guide)
- [1. Formatting (FMT)](#1-formatting-fmt)
- [2. Naming (NAM)](#2-naming-nam)
- [3. Files, packages, directories (FIL)](#3-files-packages-directories-fil)
- [4. Comments and documentation (DOC)](#4-comments-and-documentation-doc)
- [5. Class layout (CLS)](#5-class-layout-cls)
- [6. SystemVerilog language usage (LNG)](#6-systemverilog-language-usage-lng)
- [7. UVM components and phases (CMP)](#7-uvm-components-and-phases-cmp)
- [8. Factory and macros (FAC)](#8-factory-and-macros-fac)
- [9. Configuration (CFG)](#9-configuration-cfg)
- [10. Interfaces and DUT connection (IFC)](#10-interfaces-and-dut-connection-ifc)
- [11. Transactions (TRN)](#11-transactions-trn)
- [12. Sequences and stimulus (SEQ)](#12-sequences-and-stimulus-seq)
- [13. Objections and end of test (OBJ)](#13-objections-and-end-of-test-obj)
- [14. TLM and analysis connections (TLM)](#14-tlm-and-analysis-connections-tlm)
- [15. Checking, scoreboards, models (CHK)](#15-checking-scoreboards-models-chk)
- [16. Messaging (MSG)](#16-messaging-msg)
- [17. Functional coverage (COV)](#17-functional-coverage-cov)
- [18. Register layer (REG)](#18-register-layer-reg)
- [19. Tests and pass/fail (TST)](#19-tests-and-passfail-tst)
- [20. Reuse and integration (REU)](#20-reuse-and-integration-reu)
- [Appendix A — Minimal agent (excerpts)](#appendix-a--minimal-agent-excerpts)
- [Appendix B — Review checklist](#appendix-b--review-checklist)
- [Appendix C — Decisions where common practice disagrees](#appendix-c--decisions-where-common-practice-disagrees)

---

## 0. How to read and apply this guide

**Keywords.** Every rule carries **MUST**, **SHOULD** or **MAY**; a rule with several obligations
marks each. **MUST** = required; a review blocks on it. **SHOULD** = the default; deviate only with
a waiver. **MAY** = allowed, not required.

**Waivers.** A deviation from a MUST or a SHOULD is recorded in the code, directly before the code
it covers: `// <ID> waived: <reason>`, for example `// SEQ-13 waived: checks the DUT's response
time`. Other comments and tool directives (a lint or formatter switch) **MAY** stand between the
waiver and that code. A waiver covers the next statement, declaration or macro call; one that
covers more (a method, a class, a group of lines) says so in its reason. This comment is the only
durable record; a PR description or a chat reply does not replace it.

**Baseline.** The rules assume SystemVerilog IEEE 1800-2017 and UVM 1.2 or IEEE 1800.2-2017/2020
(Accellera 2020.3.1). Where these UVM versions differ, the rule says so.

**Precedence.**

1. The SystemVerilog LRM (IEEE 1800) and the UVM standard (IEEE 1800.2) always win.
2. When **creating** a new file or directory, follow this guide.
3. When **editing** an existing file that has a clear, consistent local style, match the file
   for formatting and naming (`FMT`, `NAM`). Do not restyle lines you are not otherwise changing.
   Matching a file never excuses breaking any other rule in new code.
4. Generated code (register models, `AUTOINST` blocks) and third-party code are exempt: change the
   generator input (`FIL-10`) or wrap the third-party code (`TST-8`).
5. If this guide is silent, copy the structure of the 2–3 nearest similar files.

**Instructions for AI assistants.**

- Apply rules by ID. When you deviate, add the waiver comment and name the rule in your reply.
- Produce the smallest correct change. Do not reformat, rename, or reorder unrelated code.
- Do not add comments that restate the code (`DOC-3`). Add comments for intent, contracts,
  trade-offs, and waivers only.
- Never invent helper macros or base classes. Use those that exist in the repository or plain UVM.
- Before finishing, run the checklist in [Appendix B](#appendix-b--review-checklist).

---

## 1. Formatting (`FMT`)

| ID | Rule |
|---|---|
| FMT-1 | **MUST** indent with **3 spaces** per level. No tab characters anywhere in the file. |
| FMT-2 | **MUST**, in SystemVerilog source files and file lists: ASCII only, no trailing whitespace, a single newline at the end of the file. Documentation **MAY** use UTF-8. |
| FMT-3 | **SHOULD** keep lines ≤ 120 characters. Break long argument lists one argument per line. |
| FMT-4 | **MUST** use `begin`/`end` for every `if`, `else`, `for`, `foreach`, `repeat`, `while`, `forever` body and `case` item that contains more than one statement. **SHOULD** use them even for a single statement. *Prevents dangling-`else` and "added a second line" bugs.* |
| FMT-5 | **MUST** put `begin` on the same line as its keyword; `end` lines up with the line that opened it; `end else begin` stays on one line. |
| FMT-6 | **MUST** close every `class`, `function`, `task`, `module`, `interface`, `package`, `covergroup`, `property`, `sequence`, `clocking` with an end label: `endclass : bus_driver`. *The compiler checks labels; it does not check comments.* |
| FMT-7 | **MUST NOT** put end comments on `end` (`end // while (...)`). Turn off editor auto-insertion of them. |
| FMT-8 | **MUST** put one declaration or statement per line. Where the formatter joins declarations (`FMT-13`), its layout stands. |
| FMT-9 | **SHOULD** leave one blank line between methods; **MUST NOT** leave two or more consecutive blank lines. |
| FMT-10 | **SHOULD** align consecutive declarations and assignments in columns when it aids scanning (types, names, `=`). Do not realign untouched code. |
| FMT-11 | **MUST** put one space after keywords and around binary operators: `if (a == b)`, `foreach (q[i])`. No space between a function name and `(`. Parameter lists: `uvm_driver #(bus_item)`. Where the formatter puts a space before `(` (`FMT-13`), its layout stands. |
| FMT-12 | **SHOULD** keep files under ~500 lines. A larger file usually means a class doing two jobs. |
| FMT-13 | The project **MUST** name its formatter, with its version and options (for example a checked-in flag file). Code that this formatter produces satisfies the layout rules that the formatter decides (spacing, line breaks, what shares a line) and **MUST NOT** be changed by hand. The formatter decides layout only: names, order of members, comments and every other rule still apply, and code in a region where the formatter is switched off follows the written layout rules. The project states whether indentation (`FMT-1`) and line length (`FMT-3`) are left to the formatter. **SHOULD** run the formatter on every change. *Examples with verible-verilog-format: a space before the `(` of a covergroup's `sample` and `exclude`; `timeunit` and `timeprecision` on one line.* |

```systemverilog
// Good
if (item.is_write) begin
   vif.drv_cb.wdata <= item.data;
end else begin
   vif.drv_cb.wdata <= '0;
end

// Bad: no begin/end, end comment, tab indentation
if (item.is_write)
	vif.drv_cb.wdata <= item.data;
else
	vif.drv_cb.wdata <= '0; // else
```

---

## 2. Naming (`NAM`)

### 2.1 General

| ID | Rule |
|---|---|
| NAM-1 | **MUST** use `lower_snake_case` for classes, variables, functions, tasks, instances, signals, files, packages, interfaces, modules. No `camelCase`. Names that UVM or another library defines keep their spelling. *Matches the UVM library.* |
| NAM-2 | **MUST** use `UPPER_SNAKE_CASE` for `parameter`, `localparam`, `` `define `` constants and enum literals. |
| NAM-3 | **MUST NOT** prefix anything with `uvm_` or `m_`. *`uvm_` belongs to the library; `m_` marks UVM internals (`m_sequencer`), and mixing in our own names hides that distinction.* |
| NAM-4 | **No member or argument prefixes.** Class members and method arguments use plain names. When an argument shadows a member, disambiguate with `this.`. Where the project's linter treats such shadowing as an error (for example slang `-Wshadow-value`), name the argument for what it is to the member instead (`new_cfg`, `uid_parent`, `item_name`). **MUST NOT** use a prefix or suffix that only marks an argument (`a_`, `i_`, `_arg`). An override of a virtual method keeps the argument names of the method it overrides, even where they break this rule or hide a type (`bus2reg(uvm_sequence_item bus_item, ...)`). *The LRM requires identical names; Questa rejects a renamed argument, VCS warns.* |
| NAM-5 | **SHOULD** spell words out. Abbreviate only with the approved short forms in §2.3. `config_base_seq`, not `cnfg_base_seq`. |
| NAM-6 | **SHOULD** use names that state purpose: `first_stage_timer`, not `timer1`. Short names (`i`, `idx`, `item`) are fine for small local scopes. |
| NAM-7 | **MUST** restrict UVM instance-name strings to `[a-zA-Z0-9_]`, with no double underscores. |
| NAM-8 | **MUST** pass a `create()` name string identical to the variable name: `drv = bus_driver::type_id::create("drv", this);`. In loops use `$sformatf("agent_%0d", i)`. *Log paths then map directly to code.* |
| NAM-9 | Active-low signals **MUST** end in `_n` (`rst_n`). Testbench-only nets in a module or interface **SHOULD** start with `tb_` to separate them from DUT signals. |

```systemverilog
// NAM-4: plain members, this. only when an argument shadows a member
class bus_driver extends uvm_driver #(bus_item);
   bus_agent_cfg cfg;
   int unsigned  num_driven;

   function void set_cfg(bus_agent_cfg cfg);
      this.cfg = cfg;
   endfunction : set_cfg
endclass : bus_driver
```

### 2.2 Type and instance suffixes

Class names spell the role out in full. Instance names use the short form. When several
instances of a role live in one scope, prefix the short form: `axi_agent`, `apb_agent`, `rd_ap`, `wr_ap`.

| Kind | Type name | Instance name |
|---|---|---|
| Package | `<block>_pkg` | — |
| Interface | `<block>_if` | `<block>_if_inst` (module) / `vif` (class handle) |
| Virtual interface handle | — | `vif`, or `<what>_vif` |
| Sequence item | `<block>_item` | `req`, `rsp`, `item` |
| Sequence | `<block>_<what>_seq` | `<what>_seq`, or `seq` |
| Virtual sequence | `<block>_<what>_vseq` | `vseq` |
| Sequencer | `<block>_sequencer` | `sqr` |
| Virtual sequencer | `<block>_virtual_sequencer` | `vsqr` |
| Driver | `<block>_driver` | `drv` |
| Monitor | `<block>_monitor` | `mon` |
| Agent | `<block>_agent` | `agent`, or `<block>_agent` |
| Config object | `<owner>_cfg` (`bus_agent_cfg`, `soc_env_cfg`) | `cfg`, or `<owner>_cfg` |
| Environment | `<block>_env` | `env`, or `<block>_env` |
| Scoreboard | `<block>_scoreboard` | `sb` |
| Reference model / predictor | `<block>_model`, `<block>_predictor` | `model`, `predictor` |
| Coverage collector | `<block>_coverage` | `cov` |
| Register block (generated) | `<block>_reg_block` | `regmodel` |
| Register adapter / predictor | `<bus>_reg_adapter` / `uvm_reg_predictor #(...)` | `reg_adapter` / `reg_predictor` |
| Test | `<block>_<feature>_test`, base `<block>_base_test` | — |
| Testbench top module | `<block>_tb` | DUT instance: `dut` |
| Analysis port | — | `ap`, or `<what>_ap` |
| Analysis export / imp | — | `<what>_export` / `<what>_imp` |
| Other TLM port | — | `<what>_port` |
| Constraint block | — | `<what>_cstr` |
| Covergroup / coverpoint / cross | `<what>_cg` | coverpoint `<what>_cp`, cross `<a>_x_<b>` |
| Typedef (struct, union, alias) | `<what>_t` | — |
| Enum typedef | `<what>_e` | — |
| `` `uvm_analysis_imp_decl `` suffix | `_<what>` (`_expected`, `_actual`) | — |

### 2.3 Approved short forms

`addr ap cb cfg cnt cov cp cstr data drv dut env idx if intf* mon mp pkg rd reg req rsp sb seq sqr tb vif vseq vsqr wr`
(*`intf` only where an existing codebase already uses it; new code uses `_if`.)

Protocol and standard acronyms (`axi`, `apb`, `dma`, `irq`, `crc`) are names, not abbreviations.

---

## 3. Files, packages, directories (`FIL`)

| ID | Rule |
|---|---|
| FIL-1 | **SHOULD** put one primary class, module, interface or package in each file, named after it: `bus_driver.sv` holds `class bus_driver`. Small helper classes used only by the primary class **MAY** share its file. |
| FIL-2 | **MUST** use the `.sv` extension for compiled units and files `` `include ``d into packages. **MAY** use `.svh` for macro/header files only. |
| FIL-3 | **MUST** define every class inside a package, never in a module or at compilation-unit scope. Exception: a self-contained example or unit-test file whose whole testbench is one module **MAY** declare its classes in that module. |
| FIL-4 | **MUST NOT** `import` (wildcard or explicit) at compilation-unit scope (outside a `module`, `interface` or `package`). |
| FIL-5 | Inside a package or module, `` `include "uvm_macros.svh" `` then `import uvm_pkg::*;`. For other packages **SHOULD** use explicit scope (`bus_pkg::bus_item`) or an explicit import inside the package. **SHOULD NOT** wildcard-import user packages into other packages. |
| FIL-6 | Class files are `` `include ``d into their package in dependency order. Each included file **MUST** have an include guard named after the file: `` `ifndef BUS_DRIVER_SV ``. |
| FIL-7 | **MUST NOT** use `` `timescale ``. **MUST** declare `timeunit` and `timeprecision` in every package, module and interface that contains a delay or a time value, as its first items (before any `import`). **MUST** give every delay literal a unit: `#10ns`, `#(period_ps * 1ps)`. Never a bare `#10`. *A delay is rounded to the precision of the scope that declares it, whatever unit its literal carries: `#100ps` in a class of a package with 1 ns precision is a zero delay.* Cycle delays (`##1`, `@(vif.drv_cb)`) need no unit. |
| FIL-8 | **SHOULD NOT** build "sequence library" files that collect unrelated sequences. One sequence per file under `sequences/`. |
| FIL-9 | **SHOULD** put a `README.md` in every agent, env and testbench directory: purpose, usage, config knobs, known limits. *Humans and AI both start there.* |
| FIL-10 | **MUST NOT** hand-edit generated code (register models, `AUTOINST` blocks). Change the generator input. |

**Standard agent layout**

```text
bus_agent/
   README.md
   bus_agent.f            # file list
   bus_pkg.sv             # package: types + `include of the files below
   bus_if.sv              # interface (compiled outside the package)
   bus_item.sv
   bus_agent_cfg.sv
   bus_sequencer.sv       # optional: only if it adds members
   bus_driver.sv
   bus_monitor.sv
   bus_agent.sv
   bus_coverage.sv        # optional
   bus_reg_adapter.sv     # optional
   sequences/
      bus_base_seq.sv
      bus_random_seq.sv
```

**Package skeleton**

```systemverilog
`include "uvm_macros.svh"

package bus_pkg;
   timeunit 1ns;
   timeprecision 1ps;

   import uvm_pkg::*;

   localparam int unsigned ADDR_WIDTH = 32;
   localparam int unsigned DATA_WIDTH = 32;

   typedef enum bit {BUS_READ, BUS_WRITE} bus_op_e;

   `include "bus_item.sv"
   `include "bus_agent_cfg.sv"
   `include "bus_sequencer.sv"
   `include "bus_driver.sv"
   `include "bus_monitor.sv"
   `include "bus_agent.sv"
   `include "sequences/bus_base_seq.sv"
   `include "sequences/bus_random_seq.sv"
endpackage : bus_pkg
```

---

## 4. Comments and documentation (`DOC`)

| ID | Rule |
|---|---|
| DOC-1 | **MUST** start every file with a header block: one-line summary, then what the class is for, how to use it, and its key knobs and assumptions. *This is the most valuable comment for humans and AI.* |
| DOC-2 | **SHOULD** comment **why**, not what: intent, spec references (`// Spec 4.2.1: ...`), protocol corner cases, non-obvious trade-offs, waivers. |
| DOC-3 | **MUST NOT** write comments that restate a name or the code (`// Constructor`, `// build phase`, `// Class: foo`), banner boxes around standard methods, or template leftovers (`// Insert code here`). |
| DOC-4 | **SHOULD** use `//` line comments. `/* */` is acceptable only for the file header. |
| DOC-5 | **MUST** document the public API (knobs, ports, methods others call) of reusable code (agents, envs, VIP, shared base classes) with one line above each declaration; **SHOULD** elsewhere. |
| DOC-6 | **MUST** format TODOs as `// TODO:<owner>:<P1\|P2\|P3>: <text>`, for example `// TODO:rg:P1: handle concurrent interrupts`. **MUST** resolve P1s before sign-off. *Greppable and summarizable.* |
| DOC-7 | **MUST** delete dead and commented-out code before merging. Version control remembers it. |
| DOC-8 | **MUST** state the unit of a physical or time quantity in its name or its comment: `timeout_ns`, `freq_hz`. |

```systemverilog
// SPDX-License-Identifier: Apache-2.0
// bus_driver.sv
//
// Drives bus_item requests onto bus_if through drv_cb.
// - Pulls one item at a time (just-in-time) so sequences can react to DUT state.
// - item.delay_cycles idle cycles are inserted before each request.
// - Read data and response status are written back into the same item before item_done().
// Assumes: cfg.vif set by the parent env; synchronous active-low reset.
```

---

## 5. Class layout (`CLS`)

| ID | Rule |
|---|---|
| CLS-1 | **MUST** order class contents as follows: (1) public API members (knobs, ports, handles, `rand` fields); (2) internal members, marked `local` or `protected`; (3) `` `uvm_*_utils `` registration; (4) constraints; (5) covergroups; (6) `new`; (7) UVM phase methods in phase order; (8) other public methods; (9) `local`/`protected` helper methods. A `get_type_name()` override of a parameterized class, which stands in for the name that its registration macro cannot give, **MAY** come right before `new`. |
| CLS-2 | **MUST** define methods inside the class body. **MUST NOT** use `extern` prototypes. *Maintaining two copies of every signature costs more than it gives; editors outline classes.* |
| CLS-3 | **SHOULD** mark internal state `local` (or `protected` when extensions need it). |
| CLS-4 | **MUST NOT** repeat `virtual` when overriding a UVM virtual method (`build_phase`, `do_copy`). **MUST** declare new methods that extensions may override `virtual`. |
| CLS-5 | **SHOULD** keep methods short. Split long phase bodies into named helpers (`build_agents()`, `connect_scoreboard()`). |
| CLS-6 | **SHOULD** declare variables in the narrowest scope. Do not promote a task's local to a class member. |
| CLS-7 | **SHOULD NOT** initialize variables to their default value (`int count = 0;`). |
| CLS-8 | Constructors of UVM classes: components `function new(string name, uvm_component parent)`; objects `function new(string name = "<class_name>")`. The body is `super.new(...)` plus, only if needed, construction of what the class owns: covergroups, analysis ports, child config objects (`CFG-3`), and helper objects that are part of the class and must exist before the build phase (a status object, an audit that a derived constructor fills); and the defaults of its own members (`fail_on_warnings = 1;`). **MUST NOT** create child components in `new` (`CMP-2`). Classes that do not derive from a UVM class (plain utility classes, classes of a UVM-free package) take the arguments they need. |

---

## 6. SystemVerilog language usage (`LNG`)

### 6.1 Constants and loops

| ID | Rule |
|---|---|
| LNG-1 | **MUST NOT** use magic numbers in code bodies. Name them with `localparam`/`parameter`, or derive them (`$bits`, `.size()`). |
| LNG-2 | **SHOULD** iterate with `foreach` instead of an index loop over a fixed bound. |
| LNG-3 | **SHOULD** use `logic` for 4-state variables and `bit`/`int unsigned` for 2-state testbench data. Use `wire` only where multiple drivers or the port kind require it. |
| LNG-4 | **SHOULD** compare signals that may be X/Z with `===`/`!==` in testbench code. |
| LNG-5 | **SHOULD** use sized or fill literals (`'0`, `'1`, `8'hFF`). |

### 6.2 Randomization and constraints

| ID | Rule |
|---|---|
| LNG-10 | **MUST** check every `randomize()` call and report failure with `` `uvm_fatal ``. **MUST NOT** wrap `randomize()` in `assert`. *Disabling assertions also skips the randomization.* Exception: a unit test of a randomizer or of its constraints **MAY** check an expected failure as a result. |
| LNG-11 | **SHOULD** declare variables that may ever be randomized as `rand`. **MUST NOT** fill them with `$urandom_range()` in procedural code. |
| LNG-12 | **SHOULD** prefer class constraints over inline `with {}` constraints for anything reused. Inline constraints suit test- or call-site-specific narrowing. |
| LNG-13 | **MUST** name every constraint `<what>_cstr`, scoped to what it constrains. |
| LNG-14 | Use `soft` only for simple defaults (`soft len inside {[1:64]};`). **MUST NOT** stack several soft constraints on one variable. |
| LNG-15 | **MUST** use `local::` for every variable of the calling scope inside `randomize() with {}`. *Without it, a name that also exists in the randomized object refers to the object's member.* |
| LNG-16 | **SHOULD** write clear declarative constraints first, and optimize only solver bottlenecks measured on the supported simulators. `inside` and set operators often solve faster than a `foreach` over the same values (example below). A staged solve or an element-by-element randomization of an array **MUST** keep every constraint that spans stages or elements (uniqueness, ordering, sums), and its header **MUST** say how the distribution changes. A mask **MAY** replace `%` only where both give the same result for every allowed value (an unsigned operand and a power-of-two modulus). |
| LNG-17 | **SHOULD NOT** over-randomize. When the scenario space is small, enumerate it. *Unrealistic random stimulus costs more debug time than it saves.* |

```systemverilog
// Good
constraint not_reserved_cstr {
   !(addr inside {reserved_addrs});
}

if (!item.randomize() with { addr == local::target_addr; }) begin
   `uvm_fatal("RAND", "bus_item randomization failed")
end

// Bad: assert can be disabled; foreach where inside works
assert(item.randomize());
constraint c1 { foreach (reserved_addrs[i]) addr != reserved_addrs[i]; }
```

### 6.3 Casting

| ID | Rule |
|---|---|
| LNG-20 | When a failed `$cast` would be a bug, **MUST** call `$cast` as a function and report failure with `` `uvm_fatal ``. **MUST NOT** call it as a task and rely on the simulator to stop. A cast of `null` succeeds and leaves the destination `null`: where the result must exist (the result of `clone()` or `create_object()`), **MUST** also check it for `null`. *The LRM only requires a run-time error for a failed task-form cast; simulators continue with the destination unchanged.* |
| LNG-21 | When failure is an expected outcome (type dispatch), **MUST** call `$cast` as a function inside `if` or `case (1)`, and **MUST** handle the case where nothing matches. A `null` source casts successfully and leaves the destination `null`, so check for `null` first where it is not legal. |

```systemverilog
// LNG-20: a failed cast is a bug
if (!$cast(copy, item.clone())) begin
   `uvm_fatal("TYPE", "clone of bus_item is not a bus_item")
end

// LNG-21: type dispatch
function void write(uvm_sequence_item t);
   bus_item    bus_t;
   irq_item    irq_t;
   if (t == null) begin
      `uvm_error("TYPE", "null item received")
      return;
   end
   case (1)
      $cast(bus_t, t): handle_bus(bus_t);
      $cast(irq_t, t): handle_irq(irq_t);
      default: `uvm_error("TYPE", $sformatf("unexpected type %s", t.get_type_name()))
   endcase
endfunction : write
```

### 6.4 Error checks

| ID | Rule |
|---|---|
| LNG-30 | **MUST NOT** use immediate `assert()` for procedural checks in classes. Use `if (!cond) `uvm_error(...)`. Concurrent SVA (`assert property`) belongs in interfaces and modules (`IFC-6`). |
| LNG-31 | **MUST NOT** re-check the same invariant on every iteration of a hot loop. Check once where the data enters. |
| LNG-32 | **MUST** fix all compile and elaboration warnings before merging, or waive them explicitly with a reason. |

---

## 7. UVM components and phases (`CMP`)

| ID | Rule |
|---|---|
| CMP-1 | **MUST** extend the most specific UVM base class: `uvm_driver`, `uvm_monitor`, `uvm_agent`, `uvm_env`, `uvm_scoreboard`, `uvm_subscriber`, `uvm_test`. |
| CMP-2 | **MUST** create child components in `build_phase` with the factory (`FAC-1`), with `this` as parent. |
| CMP-3 | **MUST** make TLM connections in `connect_phase`. |
| CMP-4 | **MUST** call `super.<phase>(phase)` first in every function phase you override (`build_phase`, `connect_phase`, `check_phase`, ...). In a task phase (`run_phase`, `main_phase`, ...) call `super` first when the parent is a UVM base class, or when the parent's header says that its task phase returns and is meant to be extended. A base class whose task phase never returns (a driver's `forever` loop) or starts stimulus **MUST** say so in its header (`DOC-1`), and say whether a derived class replaces or extends that phase. A phase that never returns is replaced: the derived class does not call `super`. A phase that starts stimulus and returns when it has ended **MAY** be extended, with `super` first, where its header says so. *A derived class that calls a `super.run_phase()` that never returns never reaches its own code.* |
| CMP-5 | Drivers, monitors, scoreboards, coverage and models **MUST** run in `run_phase` only and be phase-agnostic. They handle reset by watching the reset signal or a reset event, not by overriding `reset_phase`. **MUST**, while reset is active: a driver drives idle values and starts no request; an item that a reset interrupts ends with an error status and exactly one `item_done()`, and the driver does not retry it (the sequence decides; repeating a write may not be safe); a monitor publishes nothing it sampled during reset; a scoreboard flushes (`CHK-6`). Exception: a component whose job is the reset itself keeps working during reset: a driver of the clock and reset carries out reset commands, and a monitor of the reset publishes reset events and status. *Lets them be reused under any test phasing.* |
| CMP-6 | Tests (and envs, for default stimulus) **MAY** use `reset_phase`, `configure_phase`, `main_phase` and `shutdown_phase` to start sequences. **SHOULD NOT** override `pre_*`/`post_*` run-time phases; keep them as cross-domain sync points. |
| CMP-7 | **SHOULD NOT** use phase jumps. If one is unavoidable, document the cleanup each component performs. |
| CMP-8 | An agent **MUST** contain only the canonical `sqr`, `drv`, `mon` (plus protocol-intrinsic checks and coverage). It builds `sqr` and `drv` only when `get_is_active() == UVM_ACTIVE`; it always builds `mon`. |
| CMP-9 | Agents **MUST** be protocol-specific and DUT-agnostic. DUT-specific checking and coverage live outside the agent (`CHK`, `COV`). |
| CMP-10 | An agent **MUST** expose its monitor's analysis port as an agent-level handle (`ap = mon.ap;` in `connect_phase`, or in `build_phase` once the monitor has created the port); it **MUST NOT** add a second, forwarded port. *Users can connect at agent level without hidden plumbing.* |
| CMP-11 | Envs **MUST** work both as the top env and as a sub-env (`REU-1`). |
| CMP-12 | Monitors **MUST NOT** drive interface signals and **MUST NOT** block on anything except the interface. |
| CMP-13 | Drivers **SHOULD** pull items just-in-time (`get_next_item` → drive → `item_done`). Use `try_next_item` when the protocol needs idle cycles driven between items. |
| CMP-14 | Variable delays within or between items live in the item (`delay_cycles`), so sequences control timing. Only drivers, monitors and interfaces consume time for the protocol; other components (scoreboards, predictors, coverage) do not delay items. Supervision and timing utilities (watchdogs, timeouts, end-of-test polling, delay models whose purpose is time) **MAY** wait. |

---

## 8. Factory and macros (`FAC`)

| ID | Rule |
|---|---|
| FAC-1 | **MUST** create every component, sequence, sequence item and config object through the factory: `type_id::create("name"[, this])`. Never call `new` directly, except for TLM ports, exports, imps and analysis FIFOs, covergroups and `uvm_event`s. |
| FAC-2 | **MUST** register every concrete class derived from `uvm_object` (components included) with `` `uvm_component_utils `` / `` `uvm_object_utils `` (or the `_param_` variants). An abstract (`virtual`) UVM class **MAY** be registered with `` `uvm_object_abstract_utils `` / `` `uvm_component_abstract_utils `` (IEEE 1800.2-2017 and later), so that overrides can name it; it is never created without a concrete override. Under UVM 1.2, which lacks those macros, it **MUST NOT** be registered: the ordinary macros construct the class, which some simulators reject at compile time. Code that supports both registers only the concrete classes. Interface classes, plain utility classes and classes of a UVM-free package are not registered. |
| FAC-3 | Field macros (`` `uvm_field_* ``) **MAY** be used on sequence items and config objects, for print, compare, copy and record. **SHOULD** flag metadata `UVM_NOCOMPARE` and pick a radix (`UVM_HEX`) where it helps. |
| FAC-4 | **MUST NOT** use field macros in components. *They trigger implicit config-db lookups (`apply_config_settings`). Configuration is explicit here (`CFG`).* |
| FAC-5 | When field macros are not used, **MUST** implement `do_copy`, `do_compare`, `do_print`, and `do_record` if transactions are recorded. **MUST** implement `do_pack`/`do_unpack` when the item crosses DPI or a packed boundary. This applies to sequence items, transactions and config objects. A service object (a status object, a registry, a utility that components call) whose state is not data to copy or compare **MAY** implement only the hooks it supports; its header says which. |
| FAC-6 | On hot paths (a high-rate copy or compare per transaction), **SHOULD** hand-write `do_copy`/`do_compare` instead of relying on field macros, and measure. |
| FAC-7 | **MUST NOT** use the `` `uvm_do* ``, `` `uvm_create* ``, `` `uvm_send* `` or `` `uvm_rand_send* `` macro families. Write `create` → `start_item` → `randomize` → `finish_item` explicitly. |
| FAC-8 | **MUST NOT** use `` `uvm_declare_p_sequencer ``. *It locks a sequence to one sequencer type.* See `SEQ-8` for alternatives. |
| FAC-9 | **MUST** use the report macros (`` `uvm_info `` and friends), never `uvm_report_*()` calls directly. Exception: a project's own report macros **MAY** call `uvm_report_*()` when they guard the call with `uvm_report_enabled()` as the UVM macros do. *The macros skip message construction when the message is filtered.* |
| FAC-10 | Overrides **SHOULD** use `orig::type_id::set_type_override(new_t::get_type())` or `set_inst_override`, and reach the factory with `uvm_factory::get()`. *Overrides by type name are not checked at compile time.* |
| FAC-11 | **MUST NOT** use deprecated UVM features or undocumented UVM internals (members starting with `m_`, except `m_sequencer`, which is documented). |

---

## 9. Configuration (`CFG`)

| ID | Rule |
|---|---|
| CFG-1 | Each configurable component **MUST** have **one config object** (`<owner>_cfg extends uvm_object`). The config object holds `is_active`, the virtual interface, enables (`has_coverage`, `has_checks`) and protocol knobs. |
| CFG-2 | Configuration **MUST** flow **top-down by direct assignment**. The parent creates or receives the child's cfg and sets `child.cfg = ...` in its own `build_phase`, after `create` and before the child's `build_phase` runs. |
| CFG-3 | Parent config objects **MUST** contain their children's config objects as `rand` handles and construct them in `new` (`CLS-8`). One `randomize()` of the top cfg then randomizes every child, and cross-cfg constraints are possible. *A child handle that is not `rand`, or still `null`, is not randomized.* |
| CFG-4 | `uvm_config_db` is used only (a) to hand virtual interfaces from the testbench module to the test, and (b) across a reuse boundary where the parent cannot see the child type. Every `get` **MUST** use `get(this, "", "<field>", ...)` and **MUST** be checked: a mandatory value with `` `uvm_fatal `` (`CFG-6`). An optional lookup **MAY** go on when `get` finds nothing, provided it checks the handle it got and the header documents the fallback (a cfg that the parent assigned directly). |
| CFG-5 | **MUST NOT** use `uvm_resource_db` directly. **MUST NOT** call `uvm_config_db` in run-time loops. *It is slow, and it costs most at integration level.* Exception: the `REG::` resources that UVM's register sequences read (`NO_REG_TESTS`, `NO_REG_BIT_BASH_TEST`, ...) are set and read through `uvm_resource_db`, as UVM defines them. |
| CFG-6 | **MUST** validate that a mandatory cfg or vif was received early (`build_phase`/`connect_phase`) with `` `uvm_fatal ``. Never let a null handle crash later. |
| CFG-7 | Agents **MUST** override `get_is_active()` to return `cfg.is_active`. |
| CFG-8 | Env cfgs **MUST** carry `has_<agent>` bits (default 1) that gate building the agent, fetching its vif, and connecting it (`REU-2`). |
| CFG-9 | Settings that change at run time and status shared between components **MUST** live in config or status objects (handles) that the parent passes down, never re-fetched from `uvm_config_db`. |
| CFG-10 | Tests customize the cfg through base-test hooks, not by rewriting `build_phase`. The base test runs, in this order: `do_type_overrides()` (which may replace a cfg class with one that adds constraints) → cfg creation → `modify_cfg()` → checked `randomize()` (`LNG-10`) → `modify_randomized_cfg()` → env build. In `modify_cfg()` a test pins a `rand` knob with `rand_mode(0)` and an assignment, or switches a constraint with `constraint_mode()`. **MUST NOT** plainly assign a `rand` knob in `modify_cfg()`: the randomization overwrites it. `modify_randomized_cfg()` takes plain assignments; no constraint checks them, so use it only for values that no constraint relates to others. |

```systemverilog
// soc_env_cfg.sv: child cfgs are rand handles built in new (CFG-3)
class soc_env_cfg extends uvm_object;
   bit                         has_bus_agent = 1;
   rand bus_pkg::bus_agent_cfg bus_agent_cfg;
   rand dma_pkg::dma_agent_cfg dma_agent_cfg;

   `uvm_object_utils(soc_env_cfg)

   constraint dma_burst_cstr {
      dma_agent_cfg.max_burst_len <= bus_agent_cfg.max_burst_len;
   }

   function new(string name = "soc_env_cfg");
      super.new(name);
      bus_agent_cfg = bus_pkg::bus_agent_cfg::type_id::create("bus_agent_cfg");
      dma_agent_cfg = dma_pkg::dma_agent_cfg::type_id::create("dma_agent_cfg");
   endfunction : new
endclass : soc_env_cfg

// soc_single_burst_test.sv: pins a rand knob before the base test randomizes the cfg (CFG-10)
function void modify_cfg();
   cfg.bus_agent_cfg.max_burst_len.rand_mode(0);
   cfg.bus_agent_cfg.max_burst_len = 1;
endfunction : modify_cfg

// soc_env.sv: build_phase hands cfg objects down
function void build_phase(uvm_phase phase);
   super.build_phase(phase);
   if (cfg == null) begin
      `uvm_fatal("CFG", "soc_env.cfg must be set by the parent before build_phase")
   end
   if (cfg.has_bus_agent) begin
      bus_agent     = bus_pkg::bus_agent::type_id::create("bus_agent", this);
      bus_agent.cfg = cfg.bus_agent_cfg;
   end
   sb     = soc_scoreboard::type_id::create("sb", this);
   sb.cfg = cfg;
endfunction : build_phase
```

---

## 10. Interfaces and DUT connection (`IFC`)

| ID | Rule |
|---|---|
| IFC-1 | **MUST** use one SystemVerilog interface instance per DUT interface. The UVM side reaches it only through a virtual interface handle. |
| IFC-2 | **MUST** generate clocks and resets in a module (the testbench top or a clock/reset agent's interface), never in classes and never in a `program`. **MUST NOT** use `program` blocks. |
| IFC-3 | **SHOULD** use clocking blocks for synchronous signals: `drv_cb` (outputs driven, inputs sampled) and `mon_cb` (inputs only). Add modports `drv_mp`/`mon_mp` that expose the clocking blocks plus any asynchronous signals. |
| IFC-4 | Where the interface has clocking blocks and modports (`IFC-3`), drivers **MUST** access only `drv_cb`/`drv_mp` and monitors only `mon_cb`/`mon_mp`. |
| IFC-5 | Hierarchical references into the DUT (probes, internal coverage) **MUST** be confined to one bound interface or module. Classes never contain hierarchical paths. |
| IFC-6 | Low-level protocol assertions (`assert property`) and temporal coverage (`cover property`) **SHOULD** live in the interface. **MUST** label every assertion; the failure message goes through `$error`/`` `uvm_error ``. |
| IFC-7 | Non-class code **MUST** report with `$info`/`$warning`/`$error`/`$fatal`, or, when it runs with UVM, through UVM reporting (the global `uvm_report_*()` functions, or a project macro that wraps them with the guard of `FAC-9`); never `$display`. |
| IFC-8 | **SHOULD**, for emulation/acceleration readiness, put pin-level driving and sampling in interface tasks (BFM style). Keep driver and monitor classes thin and untimed except for calls into the interface. |
| IFC-9 | The testbench top `<block>_tb` **MUST** instantiate `dut`, the interfaces, the clock/reset generation and the binds. It sets the vifs into `uvm_config_db` and calls `run_test()`. It contains no other test logic. |

```systemverilog
// bus_if.sv
// Valid/ready request bus. A transfer happens at a rising edge of clk at which valid and ready are
// both 1; rdata is valid at that edge for a read. Once valid is 1, the request (write, addr, and
// wdata for a write) stays unchanged until its transfer. From the first rising edge in reset until
// the reset ends, valid is 0.
interface bus_if (input logic clk, input logic rst_n);
   timeunit 1ns;
   timeprecision 1ps;

   logic        valid;
   logic        ready;
   logic        write;
   logic [31:0] addr;
   logic [31:0] wdata;
   logic [31:0] rdata;

   // Inputs, rst_n included, are sampled just before the edge (the default input skew): the values
   // that the DUT's flip-flops take at that edge.
   clocking drv_cb @(posedge clk);
      output valid, write, addr, wdata;
      input  ready, rdata, rst_n;
   endclocking : drv_cb

   clocking mon_cb @(posedge clk);
      input valid, ready, write, addr, wdata, rdata, rst_n;
   endclocking : mon_cb

   modport drv_mp (clocking drv_cb, input rst_n);
   modport mon_mp (clocking mon_cb, input rst_n);

   valid_known_a: assert property (@(posedge clk) disable iff (!rst_n)
      !$isunknown(valid) && (valid -> !$isunknown(ready)))
      else $error("valid, or ready during valid, is X or Z");

   request_stable_a: assert property (@(posedge clk) disable iff (!rst_n)
      valid && !ready |=> valid && $stable({write, addr}) && (!write || $stable(wdata)))
      else $error("valid dropped or the request changed before ready");
endinterface : bus_if
```

---

## 11. Transactions (`TRN`)

| ID | Rule |
|---|---|
| TRN-1 | **MUST** extend `uvm_sequence_item`, never `uvm_transaction`. |
| TRN-2 | **SHOULD** use one item class per agent for both driver and monitor. Response fields (read data, status) are non-`rand` members of the same item. |
| TRN-3 | Fields that could be randomized **SHOULD** be `rand` (`LNG-11`). Metadata (timestamps, IDs, debug) **MUST NOT** be `rand` and **MUST** be excluded from compare (`UVM_NOCOMPARE`, or skipped in `do_compare`). |
| TRN-4 | Constraints in an item **MUST** define **legality** only. Scenario shaping belongs in sequences. |
| TRN-5 | **SHOULD** provide a short one-line `convert2string()` for log messages. Full dumps use `sprint()` (`MSG-7`). |
| TRN-6 | A monitor **MUST** check every field that the protocol defines as valid at the sampling point for X or Z (`$isunknown`) before storing it in a 2-state field, and report an unknown with `` `uvm_error ``; or it keeps the field 4-state until it is checked. A negative test that expects unknowns demotes that error by ID (`MSG-9`). A component that returns sampled values to a caller (a driver writing read data into the item) **MUST** return a status other than `UVM_IS_OK` for an unknown value (`UVM_HAS_X`). The same holds at every boundary where a 4-state value becomes 2-state: a consumer of an item with a status **MUST** check the status before it converts the value. A monitor that drops a transfer for an unknown value means its consumers never see it; a negative test that provokes this **SHOULD** tell them not to expect it, so that the run still ends by its normal mechanism and not by the timeout (`OBJ-2`, `OBJ-7`). *Assigning `4'b10x1` to a `bit [3:0]` stores `4'b1001`: the scoreboard can pass on a value the DUT never drove.* |

```systemverilog
class bus_item extends uvm_sequence_item;
   rand bus_op_e                 op;
   rand bit [ADDR_WIDTH-1:0]     addr;
   rand bit [DATA_WIDTH-1:0]     data;
   rand int unsigned             delay_cycles;
   // Set by the driver; checked by the starter of the item (REG-7 for register accesses), not by compare.
   uvm_status_e                  status;

   `uvm_object_utils_begin(bus_item)
      `uvm_field_enum(bus_op_e, op, UVM_DEFAULT)
      `uvm_field_int(addr, UVM_DEFAULT | UVM_HEX)
      `uvm_field_int(data, UVM_DEFAULT | UVM_HEX)
      `uvm_field_int(delay_cycles, UVM_DEFAULT | UVM_NOCOMPARE | UVM_DEC)
      `uvm_field_enum(uvm_status_e, status, UVM_DEFAULT | UVM_NOCOMPARE)
   `uvm_object_utils_end

   constraint addr_aligned_cstr {
      addr[1:0] == 2'b00;
   }

   constraint delay_cycles_cstr {
      soft delay_cycles inside {[0:MAX_DELAY_CYCLES]};
   }

   function new(string name = "bus_item");
      super.new(name);
   endfunction : new

   function string convert2string();
      return $sformatf("%s addr=0x%0h data=0x%0h", op.name(), addr, data);
   endfunction : convert2string
endclass : bus_item
```

---

## 12. Sequences and stimulus (`SEQ`)

| ID | Rule |
|---|---|
| SEQ-1 | Stimulus lives in sequences and virtual sequences, even when only one test uses it. A test **SHOULD** only configure, select and start sequences and state what it expects; scenario code (loops over sequences, `fork` of sequences, waits on DUT events) belongs in a virtual sequence. *A scenario in a sequence can be started from another test, a virtual sequence or a higher level; one written in a test cannot.* |
| SEQ-2 | **MUST** generate items with the explicit pattern: `create` → `start_item` → `randomize` (late randomization, after `start_item`) → `finish_item`. A sequence that sets fields of an item after randomizing it **MUST** keep the item's constraints, those of a factory override included: it randomizes with the values as inline constraints (`LNG-15`), or its header says which constraints it does not keep. |
| SEQ-3 | **MUST** `randomize()` every sequence object before `start()`, even if it has no `rand` members. *A factory override may add some.* |
| SEQ-4 | Leaf sequences **MUST** be started on a real sequencer, with the parent sequence passed when nested: `seq.start(sqr, this)`. *Needed for locks, grabs, priorities, interrupt handling and sequencer-scoped config.* |
| SEQ-5 | Virtual sequences **MUST** be started on the env's virtual sequencer if it has one, otherwise on `null`. They receive sub-sequencer handles as members set before `start()`. |
| SEQ-6 | Register accesses inside sequences **MUST** pass `.parent(this)`. |
| SEQ-7 | Sequences are not phase-aware. They **MUST** run correctly when started from any run-time phase. |
| SEQ-8 | A sequence that needs data beyond its sequencer gets it from (a) members set by the starter, or (b) its cfg object set before start. **MUST NOT** use `p_sequencer` (`FAC-8`). |
| SEQ-9 | **SHOULD** use control knobs: `rand` members of the sequence (`num_items`, `burst_len`) that `body()` uses and the starter constrains. |
| SEQ-10 | **MUST** put housekeeping (objections, locks) in `pre_start`/`post_start`, never in `pre_body`/`post_body`; `body()` holds only behaviour. A sequence that holds a lock, a grab or an objection **MUST** also release it when it is killed (`do_kill()`; a kill skips `post_start`) and when a reset ends its work (`OBJ-6`). |
| SEQ-11 | **MUST NOT** use `default_sequence`. **MAY** use `uvm_sequence_library` for random selection among sequences. |
| SEQ-12 | Directed bug-reproduction sequences **SHOULD** be named after the bug: `bus_bug1234_seq`. |
| SEQ-13 | **SHOULD** synchronize sequences with `uvm_event`/`uvm_barrier` handles passed via cfg; **SHOULD NOT** poll signals from sequences. |
| SEQ-14 | **MUST NOT** override `pre_do`/`mid_do`/`post_do`, except to change a sequence you cannot or should not edit, together with a factory override. |
| SEQ-15 | Layered protocols **SHOULD** use one sequencer per layer, each with its own item type. Translation (layering) sequences on each sequencer move items between layers; drivers and monitors stay at the lowest layer and stay thin. |

```systemverilog
class bus_random_seq extends uvm_sequence #(bus_item);
   rand int unsigned num_items;

   `uvm_object_utils(bus_random_seq)

   constraint num_items_cstr {
      soft num_items inside {[1:MAX_SEQ_ITEMS]};
   }

   function new(string name = "bus_random_seq");
      super.new(name);
   endfunction : new

   task body();
      repeat (num_items) begin
         req = bus_item::type_id::create("req");
         start_item(req);
         if (!req.randomize()) begin
            `uvm_fatal("RAND", "bus_item randomization failed")
         end
         finish_item(req);
      end
   endtask : body
endclass : bus_random_seq

class soc_traffic_vseq extends uvm_sequence;
   uvm_sequencer_base bus_sqr;   // set by the starter before start()
   uvm_sequencer_base dma_sqr;

   `uvm_object_utils(soc_traffic_vseq)

   function new(string name = "soc_traffic_vseq");
      super.new(name);
   endfunction : new

   task body();
      bus_random_seq bus_seq = bus_random_seq::type_id::create("bus_seq");
      dma_random_seq dma_seq = dma_random_seq::type_id::create("dma_seq");
      if (!bus_seq.randomize() || !dma_seq.randomize()) begin
         `uvm_fatal("RAND", "child sequence randomization failed")
      end
      fork
         bus_seq.start(bus_sqr, this);
         dma_seq.start(dma_sqr, this);
      join
   endtask : body
endclass : soc_traffic_vseq
```

---

## 13. Objections and end of test (`OBJ`)

| ID | Rule |
|---|---|
| OBJ-1 | The test **MUST** raise **one** objection around its top-level stimulus in each run-time phase it uses and drop it when the stimulus completes. |
| OBJ-2 | Completion of in-flight traffic is **event-driven**: a scoreboard (or watchdog) holds an objection while it has outstanding expected items and drops it when they drain. **MUST NOT** wait with fixed delays (`#10us`) at the end of a test. |
| OBJ-3 | **SHOULD NOT** raise or drop objections per transaction in hot paths. If a component must, it raises once when its queue goes non-empty and drops once when it empties. |
| OBJ-4 | **MUST** pass a description string to every `raise_objection`/`drop_objection`. *Shown by `+UVM_OBJECTION_TRACE`.* |
| OBJ-5 | **MAY** call `set_propagate_mode(0)` on phase objections in the base test, after checking that nothing relies on the objection totals or callbacks of intermediate components. *Hierarchical propagation costs simulation time.* |
| OBJ-6 | Sequences that object **MUST** do so in `pre_start`/`post_start`, guarded by `get_starting_phase() != null`, and drop in `do_kill` if they can be killed. |
| OBJ-7 | **MUST** set a global timeout (`uvm_root::set_timeout` or a watchdog component) as a safety net; **MUST NOT** use it as the end-of-test mechanism. |

---

## 14. TLM and analysis connections (`TLM`)

| ID | Rule |
|---|---|
| TLM-1 | Components **MUST** communicate through TLM ports, exports and imps, not through direct handles to each other. Exceptions: a test or virtual sequence referencing an agent's sequencer; a parent setting `child.cfg`; a status object or a service (a register model, an objection aggregator, a memory model) whose handle the parent passes down. Streams of transactions always go through TLM. |
| TLM-2 | **MUST** use analysis ports for monitor → subscriber traffic (one-to-many, may be unconnected). |
| TLM-3 | **SHOULD** connect port → export directly. When buffering is needed, the `uvm_tlm_analysis_fifo` **SHOULD** be inside the consuming component. |
| TLM-4 | A component with several inputs **MUST** use `` `uvm_analysis_imp_decl(_<what>) `` (declared at package scope) with descriptive suffixes (`_expected`, `_actual`), not generic `_imp1`. |
| TLM-5 | Analysis `write()` implementations **MUST NOT** consume time and **MUST NOT** modify the received object or any object it refers to. Clone first if you need to store and mutate it; the clone copies every object that the receiver will change (a deep copy where it matters). |
| TLM-6 | A publisher **MUST** create a new object for every `write()` and **MUST NOT** change an object after publishing it. A subscriber **MAY** then keep the handle as it is; it clones only to change the object (`TLM-5`). Values of a non-class type (a struct, an integer) are copies and are exempt. |

---

## 15. Checking, scoreboards, models (`CHK`)

| ID | Rule |
|---|---|
| CHK-1 | **SHOULD** put each check where it is simplest: signal-level protocol checks in SVA (`IFC-6`); end-to-end data checks in scoreboards and models; simple read-after-write checks in the sequence or test. |
| CHK-2 | Scoreboards and models **MUST** live in the env, outside agents, and be fed by analysis connections. |
| CHK-3 | For simple in-order flows **SHOULD** use `uvm_in_order_comparator` (or an equivalent shared component) instead of a hand-rolled one. |
| CHK-4 | Store expected items in structures matched to the lookup: queues for in-order, associative arrays keyed by ID or address for out-of-order. **MUST NOT** do linear searches over large queues per transaction. |
| CHK-5 | **MUST** report leftovers in `check_phase`: expected items never matched, and actual items never expected. |
| CHK-6 | Scoreboards **MUST** flush state on reset and drop any objection they hold. |
| CHK-7 | **SHOULD** handle interrupts with a dedicated handler component that detects the interrupt and starts an ISR sequence. The ISR sequence, or a supervising sequence that starts it as its child, **MUST** own the locks or grabs: a lock admits only its owner and the owner's children. **MUST** take locks on several sequencers in one fixed order, and release them when the ISR completes, is killed, or a reset ends it (`SEQ-10`). A lock does not interrupt an item already being driven; the handler's header **MUST** state the service latency that allows. *This only works if every sequence runs on a sequencer (`SEQ-4`).* |

---

## 16. Messaging (`MSG`)

| ID | Rule |
|---|---|
| MSG-1 | UVM classes report only with `` `uvm_info ``/`` `uvm_warning ``/`` `uvm_error ``/`` `uvm_fatal `` (or the `_context` variants), or through a project's own report macros that guard the call as these do (`FAC-9`). Classes of a UVM-free package report with the severity tasks `$info`/`$warning`/`$error`/`$fatal`. **MUST NOT** use `$display` in classes. |
| MSG-2 | Message ID: a static, functional ID (`"AXI_RD_RSP"`, `"SB_MISMATCH"`), or `get_type_name()` when nothing more specific fits. **MUST NOT** use `get_name()`. *Instance names depend on where the class is instantiated and are already printed in the message header.* |
| MSG-3 | **SHOULD** use the verbosity levels as follows: `UVM_NONE`, almost never; `UVM_LOW`, rare test-level milestones; `UVM_MEDIUM`, the default for transaction-level activity; `UVM_HIGH`/`UVM_DEBUG`, testbench internals and tracing. |
| MSG-4 | **MUST** use the severities as follows: `UVM_INFO`, information; `UVM_WARNING`, suspicious but possibly legal; `UVM_ERROR`, a definite failure where simulation can usefully continue; `UVM_FATAL`, simulation cannot meaningfully continue (missing cfg, randomize failure, broken environment). |
| MSG-5 | Remember that envs are reused vertically. A message per packet at `UVM_MEDIUM` from every block floods full-chip logs. **SHOULD** print whole packets at `UVM_HIGH` or above. |
| MSG-6 | **MUST** guard expensive message construction (loops, large `sprint()`) with `if (uvm_report_enabled(UVM_HIGH, UVM_INFO, "<ID>")) begin ... end`. **MUST NOT** guard with `get_report_verbosity_level()`, which ignores ID-level settings. |
| MSG-7 | **SHOULD** print objects with `sprint()` so the active printer policy controls the format. `convert2string()` is for one-line summaries only (`TRN-5`). |
| MSG-8 | Messages **SHOULD** say what was expected, what was observed, and where: `"rdata mismatch addr=0x%0h exp=0x%0h act=0x%0h"`. |
| MSG-9 | A passing test produces **no** `UVM_ERROR` or `UVM_FATAL`. A negative test **MUST** catch each expected error with a report catcher added to the component that reports it, which matches its ID and the severity it has when it reaches the catcher (no catcher that UVM consults earlier may change that severity for that ID), counts the matches itself, and demotes only those. **MUST** fail when that count differs from the expected count (check it in `check_phase`), also for a match that arrives after the check, and **MUST** still fail on any other error. **MAY** use the report server's per-ID count instead only when nothing else can report that ID and the test subtracts the count it had at its start. *The per-ID count ignores source and severity: an info message with the same ID elsewhere satisfies it.* |

```systemverilog
if (uvm_report_enabled(UVM_HIGH, UVM_INFO, "SB_QUEUE")) begin
   foreach (expected_q[i]) begin
      `uvm_info("SB_QUEUE", $sformatf("expected_q[%0d]: %s", i, expected_q[i].convert2string()), UVM_HIGH)
   end
end
```

```systemverilog
// Catches one expected error of the component it is added to, counts it and demotes it (MSG-9).
class expected_error_catcher extends uvm_report_catcher;
   string       expected_id;
   uvm_severity expected_severity = UVM_ERROR;
   int unsigned count;

   `uvm_object_utils(expected_error_catcher)

   function new(string name = "expected_error_catcher");
      super.new(name);
   endfunction : new

   function action_e catch();
      if (get_id() == expected_id && get_severity() == expected_severity) begin
         count++;
         set_severity(UVM_INFO);
      end
      return THROW;
   endfunction : catch
endclass : expected_error_catcher

// In the negative test:
function void connect_phase(uvm_phase phase);
   super.connect_phase(phase);
   catcher             = expected_error_catcher::type_id::create("catcher");
   catcher.expected_id = "BUS_X";
   uvm_report_cb::add(env.bus_agent.mon, catcher);   // only the monitor's messages reach it
endfunction : connect_phase

function void check_phase(uvm_phase phase);
   super.check_phase(phase);
   if (catcher.count != expected_x_reads) begin
      `uvm_error("TEST_EXPECT", $sformatf("expected %0d BUS_X errors, got %0d", expected_x_reads, catcher.count))
   end
endfunction : check_phase
```

---

## 17. Functional coverage (`COV`)

| ID | Rule |
|---|---|
| COV-1 | Covergroups **MUST** live in coverage components (`uvm_subscriber` or a component with `` `uvm_analysis_imp_decl `` inputs). They are constructed in `new()`. Never put covergroups in items, sequences or monitors. |
| COV-2 | **MUST** sample explicitly with `cg.sample()` (or a `with function sample(...)` covergroup), not with a clock event. |
| COV-3 | **SHOULD** sample protocol-valid observations at the DUT boundary, meaning accepted requests and completed responses as monitors publish them, plus required internal state through a bound interface. **MUST NOT** count generated or queued stimulus as evidence that the DUT exercised a scenario. Coverage of what a generator intended, if kept, lives in its own covergroups, apart from verification-plan coverage. |
| COV-4 | **SHOULD** keep specification coverage and implementation coverage in separate covergroups. |
| COV-5 | **MUST** gate coverage collection with `cfg.has_coverage`. |
| COV-6 | Exclude values with `ignore_bins`. **MUST NOT** use `illegal_bins`; illegal values are errors that assertions or checkers must report. |
| COV-7 | **SHOULD** set `option.at_least` above 1 where repeated hits matter. **MUST NOT** set `option.weight` or `option.goal`. Exception: weight 0 on coverpoints that exist only to build a cross, so that the cross alone counts. *A goal under 100 hides holes, and weights turn the total into a negotiation.* |
| COV-8 | **SHOULD** design bins from the verification plan: typical, boundary and special values. Keep `iff` guards simple. |
| COV-9 | **SHOULD** use `cover property` in interfaces for temporal protocol coverage. |

---

## 18. Register layer (`REG`)

| ID | Rule |
|---|---|
| REG-1 | **MUST** generate the register model; never hand-edit it (`FIL-10`). |
| REG-2 | Each env that uses registers **MUST** have a `regmodel` handle. The env builds the model only when `regmodel == null` (top level); otherwise the parent assigns the sub-block. |
| REG-3 | The register hierarchy **SHOULD** mirror the DUT hierarchy. Child blocks are named after their agent or env. |
| REG-4 | **MUST** use explicit prediction: a `uvm_reg_predictor` connected to the bus monitor, with `set_auto_predict(0)`. *Auto-prediction misses accesses that do not go through the register model.* |
| REG-5 | Each bus agent that carries register traffic provides a `<bus>_reg_adapter`. **MUST** set `provides_responses = 0` when the driver writes read data and status into the request before `item_done()`, as `bus_driver` in Appendix A does. **MUST** set it to 1 only when the driver sends a separate response item (`item_done(rsp)` or `put_response(rsp)`), and that response **MUST** carry the request's IDs (`rsp.set_id_info(req)`). *With `provides_responses = 1` and no separate response, every register access waits forever.* |
| REG-6 | Register sequences **MUST** extend `uvm_sequence` (or a shared register base sequence), have a `regmodel` member set before `start()`, and pass `.parent(this)` on every access. |
| REG-7 | **MUST** check the returned `uvm_status_e` of every register access. |

---

## 19. Tests and pass/fail (`TST`)

| ID | Rule |
|---|---|
| TST-1 | The test hierarchy **SHOULD** have at most four levels: `uvm_test` → project/common base test (reporting, printer, timeout, objection policy) → `<block>_base_test` (builds the env and cfg, runs reset and init) → `<block>_<feature>_test`. Prefer composing sequences over deep test inheritance. |
| TST-2 | The base test **MUST** expose hooks: `virtual function void do_type_overrides()`, `virtual function void modify_cfg()`, `virtual function void modify_randomized_cfg()` (`CFG-10`), and `virtual function uvm_sequence_base create_stimulus()`, whose sequence the base test starts in `main_phase` under its objection. Concrete tests **SHOULD** override hooks, not `build_phase` or the base test's phase tasks (`CMP-4`). |
| TST-3 | The env **MUST** run legal default traffic under an empty test, or the base test **MUST** provide a smoke test. |
| TST-4 | Select tests with `+UVM_TESTNAME` and `run_test()` with no argument. Prefer plusargs to recompiling for knobs. Exceptions: a top that must also run without UVM's command-line DPI (`UVM_NO_DPI`) **MAY** read `+UVM_TESTNAME` itself and pass it to `run_test()`; a self-contained example **MAY** pass its own test name. |
| TST-5 | Tests **MUST NOT** reach deep into the hierarchy with long hard-coded paths. Use env-provided handles or accessors. |
| TST-6 | The pass criterion **MUST** be **no errors** **and** a positive check that the expected work happened (data compared, scoreboard drained with a non-zero match count). "No errors" alone does not count as a pass. A run also fails on any `$error` or `$fatal` from non-class code (SVA failures included), a simulator error, an abnormal end, or a timeout; the regression **MUST** check these, because UVM's report counts do not include them. |
| TST-7 | A test of a DUT **MUST NOT** pass or fail on message or packet counts alone; it proves that the stimulus reached the DUT through coverage. A unit test of verification code checks its results directly. |
| TST-8 | **SHOULD** wrap third-party VIP behind a thin vendor-neutral adapter (cfg, item, sequences). Vendor-specific code stays inside the wrapper. |

---

## 20. Reuse and integration (`REU`)

| ID | Rule |
|---|---|
| REU-1 | **MUST** design every env for vertical reuse from day one: no assumptions that it is the top, no global state, all external dependencies passed in through cfg. |
| REU-2 | Integration **SHOULD** de-duplicate agents through cfg `has_<agent>` flags. To keep consumers fed when an agent is turned off, the env routes every agent stream through an env-level `uvm_analysis_export`: agent → env export → consumers. The env also offers env-level analysis ports for external connection. The integrating env then connects the single remaining agent to every env export. |
| REU-3 | **MUST** create env-level exports only when at least one consumer exists (gate on the consumer's `has_*` flag). *An export with no imp behind it is a connection error.* |
| REU-4 | **MUST** adapt reused components with factory overrides and cfg, never by editing or copying their source. |
| REU-5 | Chip-specific packages extend or parameterize generic VIP. They **MUST NOT** fork it. |
| REU-6 | Parameterized agents **SHOULD** offer a `typedef` per concrete use: `typedef bus_agent #(.DATA_WIDTH(64)) bus64_agent_t;`. |

---

## Appendix A — Minimal agent (excerpts)

These files show the rules at work on a small valid/ready bus. To stay short they leave out the
license line, the include guard and most member comments (`DOC-1`, `FIL-6`, `DOC-5`), and they
rely on a base test, an env and a testbench top that are only described. For a complete UVM
testbench that compiles and runs on Questa and VCS, see the
[reference testbench of hexk-sv-lib](https://github.com/hexkDotCom/hexk-sv-lib/tree/v0.11.0/tests/reference);
it builds on that library's base classes.

```systemverilog
// bus_agent_cfg.sv
// Configuration for one bus_agent instance. Owned and set by the parent env.
class bus_agent_cfg extends uvm_object;
   uvm_active_passive_enum is_active = UVM_ACTIVE;
   bit                     has_coverage = 1;
   virtual bus_if          vif;

   `uvm_object_utils_begin(bus_agent_cfg)
      `uvm_field_enum(uvm_active_passive_enum, is_active, UVM_DEFAULT)
      `uvm_field_int(has_coverage, UVM_DEFAULT)
   `uvm_object_utils_end

   function new(string name = "bus_agent_cfg");
      super.new(name);
   endfunction : new
endclass : bus_agent_cfg
```

```systemverilog
// bus_driver.sv
// Drives bus_item requests on bus_if; writes read data and status back into the item.
// Each request goes on the bus at a rising edge, after item.delay_cycles idle cycles; at least one
// idle cycle separates two requests.
// Reset (CMP-5): no request starts while rst_n is low. A reset during an item ends it at the first
// rising edge that sees the reset: valid goes low, item.status stays UVM_NOT_OK, and item_done() is
// called once. The item is not retried; the sequence decides, since repeating a write may not be safe.
// A read whose rdata has an X or Z ends with status UVM_HAS_X; bus_monitor reports it (TRN-6).
// item.data of a read is valid only when item.status is UVM_IS_OK.
class bus_driver extends uvm_driver #(bus_item);
   bus_agent_cfg  cfg;

   local virtual bus_if vif;

   `uvm_component_utils(bus_driver)

   function new(string name, uvm_component parent);
      super.new(name, parent);
   endfunction : new

   function void connect_phase(uvm_phase phase);
      super.connect_phase(phase);
      vif = cfg.vif;
      if (vif == null) begin
         `uvm_fatal("CFG", "bus_agent_cfg.vif is null")
      end
   endfunction : connect_phase

   task run_phase(uvm_phase phase);
      drive_idle();
      forever begin
         seq_item_port.get_next_item(req);
         drive_item(req);
         seq_item_port.item_done();
      end
   endtask : run_phase

   local task drive_idle();
      vif.drv_cb.valid <= 1'b0;
   endtask : drive_idle

   local task drive_item(bus_item item);
      item.status = UVM_NOT_OK;
      @(vif.drv_cb iff vif.drv_cb.rst_n === 1'b1);
      repeat (item.delay_cycles) begin
         @(vif.drv_cb);
         if (vif.drv_cb.rst_n !== 1'b1) begin
            return;
         end
      end
      vif.drv_cb.valid <= 1'b1;
      vif.drv_cb.write <= (item.op == BUS_WRITE);
      vif.drv_cb.addr  <= item.addr;
      vif.drv_cb.wdata <= item.data;
      do begin
         @(vif.drv_cb);
         if (vif.drv_cb.rst_n !== 1'b1) begin
            drive_idle();
            return;
         end
      end while (vif.drv_cb.ready !== 1'b1);
      item.status = UVM_IS_OK;
      if (item.op == BUS_READ) begin
         logic [DATA_WIDTH-1:0] rdata = vif.drv_cb.rdata;
         item.data = rdata;
         if ($isunknown(rdata)) begin
            item.status = UVM_HAS_X;
         end
      end
      drive_idle();
   endtask : drive_item
endclass : bus_driver
```

```systemverilog
// bus_monitor.sv
// Publishes every completed bus transfer on ap. Passive; never drives the interface.
// Publishes nothing sampled while rst_n is low (CMP-5). A transfer with an X or Z in a field that is
// valid in it (write, addr, and wdata or rdata) is reported as BUS_X and not published (TRN-6).
class bus_monitor extends uvm_monitor;
   bus_agent_cfg                 cfg;
   uvm_analysis_port #(bus_item) ap;

   local virtual bus_if vif;

   `uvm_component_utils(bus_monitor)

   function new(string name, uvm_component parent);
      super.new(name, parent);
      ap = new("ap", this);
   endfunction : new

   function void connect_phase(uvm_phase phase);
      super.connect_phase(phase);
      vif = cfg.vif;
      if (vif == null) begin
         `uvm_fatal("CFG", "bus_agent_cfg.vif is null")
      end
   endfunction : connect_phase

   task run_phase(uvm_phase phase);
      forever begin
         @(vif.mon_cb iff (vif.mon_cb.rst_n === 1'b1 && vif.mon_cb.valid === 1'b1 && vif.mon_cb.ready === 1'b1));
         publish_transfer();
      end
   endtask : run_phase

   local function void publish_transfer();
      bus_item               item  = bus_item::type_id::create("item");
      logic                  write = vif.mon_cb.write;
      logic [ADDR_WIDTH-1:0] addr  = vif.mon_cb.addr;
      logic [DATA_WIDTH-1:0] data  = write ? vif.mon_cb.wdata : vif.mon_cb.rdata;
      if ($isunknown({write, addr, data})) begin
         `uvm_error("BUS_X", $sformatf("X or Z in a transfer: write=%b addr=0x%h data=0x%h", write, addr, data))
         return;
      end
      item.op   = write ? BUS_WRITE : BUS_READ;
      item.addr = addr;
      item.data = data;
      ap.write(item);
   endfunction : publish_transfer
endclass : bus_monitor
```

```systemverilog
// bus_agent.sv
// Canonical active/passive agent: sqr + drv when active, mon always.
// Connect consumers to agent.ap (the monitor's port).
class bus_agent extends uvm_agent;
   bus_agent_cfg                 cfg;
   uvm_sequencer #(bus_item)     sqr;
   bus_driver                    drv;
   bus_monitor                   mon;
   uvm_analysis_port #(bus_item) ap;

   `uvm_component_utils(bus_agent)

   function new(string name, uvm_component parent);
      super.new(name, parent);
   endfunction : new

   function void build_phase(uvm_phase phase);
      super.build_phase(phase);
      if (cfg == null) begin
         `uvm_fatal("CFG", "bus_agent.cfg must be set by the parent before build_phase")
      end
      mon     = bus_monitor::type_id::create("mon", this);
      mon.cfg = cfg;
      if (get_is_active() == UVM_ACTIVE) begin
         sqr     = uvm_sequencer #(bus_item)::type_id::create("sqr", this);
         drv     = bus_driver::type_id::create("drv", this);
         drv.cfg = cfg;
      end
   endfunction : build_phase

   function void connect_phase(uvm_phase phase);
      super.connect_phase(phase);
      ap = mon.ap;
      if (get_is_active() == UVM_ACTIVE) begin
         drv.seq_item_port.connect(sqr.seq_item_export);
      end
   endfunction : connect_phase

   function uvm_active_passive_enum get_is_active();
      return cfg.is_active;
   endfunction : get_is_active
endclass : bus_agent
```

```systemverilog
// bus_reg_adapter.sv
// Converts register accesses to bus_item and back. bus_driver writes read data and status into the
// request before item_done(), so provides_responses is 0 (REG-5).
class bus_reg_adapter extends uvm_reg_adapter;
   `uvm_object_utils(bus_reg_adapter)

   function new(string name = "bus_reg_adapter");
      super.new(name);
      provides_responses = 0;
   endfunction : new

   function uvm_sequence_item reg2bus(const ref uvm_reg_bus_op rw);
      bus_item item = bus_item::type_id::create("item");
      item.op   = (rw.kind == UVM_WRITE) ? BUS_WRITE : BUS_READ;
      item.addr = rw.addr;
      item.data = rw.data;
      return item;
   endfunction : reg2bus

   // The argument keeps the name UVM gives it (bus_item), which hides the class bus_item here; the
   // package scope names the class.
   function void bus2reg(uvm_sequence_item bus_item, ref uvm_reg_bus_op rw);
      bus_pkg::bus_item item;
      if (bus_item == null || !$cast(item, bus_item)) begin
         `uvm_fatal("TYPE", "bus2reg expects a bus_item")
      end
      rw.kind   = (item.op == BUS_WRITE) ? UVM_WRITE : UVM_READ;
      rw.addr   = item.addr;
      rw.data   = item.data;
      rw.status = item.status;
   endfunction : bus2reg
endclass : bus_reg_adapter
```

```systemverilog
// bus_base_test.sv (excerpt)
   // Hook (TST-2): the stimulus of the test, started in main_phase; null starts none.
   virtual function uvm_sequence_base create_stimulus();
      return null;
   endfunction : create_stimulus

   task main_phase(uvm_phase phase);
      uvm_sequence_base seq = create_stimulus();
      super.main_phase(phase);
      if (seq != null) begin
         phase.raise_objection(this, "stimulus");
         seq.start(env.bus_agent.sqr);
         phase.drop_objection(this, "stimulus done");
      end
   endtask : main_phase
```

```systemverilog
// bus_smoke_test.sv
// Smoke test: 20 random transfers. bus_base_test starts them in main_phase under its objection;
// the scoreboard holds its own objection until its expected items drain (OBJ-2).
class bus_smoke_test extends bus_base_test;
   `uvm_component_utils(bus_smoke_test)

   function new(string name, uvm_component parent);
      super.new(name, parent);
   endfunction : new

   function uvm_sequence_base create_stimulus();
      bus_random_seq seq = bus_random_seq::type_id::create("seq");
      if (!seq.randomize() with { num_items == 20; }) begin
         `uvm_fatal("RAND", "bus_random_seq randomization failed")
      end
      return seq;
   endfunction : create_stimulus
endclass : bus_smoke_test
```

---

## Appendix B — Review checklist

Run this on every change, whether you are a human reviewer or an AI before handing back code.

- [ ] 3-space indent, no tabs; `begin`/`end` on bodies of more than one statement (SHOULD on single statements); end labels on the constructs `FMT-6` lists; lines ≤ 120 (SHOULD); ASCII in SV source; the formatter's layout kept (`FMT-13`) (`FMT`)
- [ ] Names: `lower_snake`, constants `UPPER`, no `m_`/`uvm_` prefixes, suffixes per §2.2, `create()` string = variable name; overrides keep UVM's argument names (`NAM`)
- [ ] File header present; public API of reusable code documented; no restating comments; no dead code; TODOs formatted (`DOC`)
- [ ] Classes in packages (a self-contained example or unit-test file excepted, `FIL-3`); no `extern`; no compilation-unit imports; no `` `timescale ``; `timeunit`/`timeprecision` first in every package, module and interface with timing; all delays carry units (`FIL`, `CLS`)
- [ ] Every `randomize()` checked with `` `uvm_fatal `` (a randomizer's unit test may expect a failure); no `assert(randomize())`; `local::` in inline constraints; constraints named `_cstr`; `$cast` checked as a function (`LNG`)
- [ ] Concrete UVM classes registered (`FAC-2`); components, sequences, items and config objects created through the factory (exceptions in `FAC-1`); copy, compare and print hooks on items and cfgs (`FAC-5`); no `` `uvm_do* ``; no `p_sequencer`; no field macros in components (`FAC`)
- [ ] cfg handed down by the parent; child cfgs `rand` and built in `new`; mandatory cfg/vif checked early with `` `uvm_fatal ``; no `uvm_config_db` in run-time loops (`CFG`)
- [ ] Leaf sequences on a sequencer with parent `this`; virtual sequences on the virtual sequencer or `null`; sequences randomized before `start()`; locks and objections released on kill and reset; `.parent(this)` and status checks on register accesses (`SEQ`, `REG`)
- [ ] Drivers idle and end interrupted items on reset; monitors publish nothing during reset and check valid fields for X/Z; drivers return a non-OK status for unknown read data; drivers, monitors and scoreboards use `run_phase` only (`CMP`, `TRN`)
- [ ] A new object per `write()`; `write()` takes no time and changes nothing it receives (`TLM`)
- [ ] One test objection per run-time phase, with descriptions; no fixed end-of-test delays; a global timeout set (`OBJ`)
- [ ] Message IDs static (never `get_name()`); severity and verbosity per `MSG-3`/`MSG-4`; expensive prints guarded; negative tests count their expected errors by source, ID and severity (`MSG`)
- [ ] Scoreboard reports leftovers in `check_phase`; passing needs positive evidence; `$error`, SVA failures and timeouts fail the run (`CHK`, `TST`)
- [ ] Zero new compile or elaboration warnings, or each one waived with a reason (`LNG-32`)
- [ ] Every deviation from a MUST or SHOULD carries a `// <ID> waived: <reason>` comment (§0)

---

## Appendix C — Decisions where common practice disagrees

These are deliberate choices. Change one only by editing this table and the rules it points to.

| Topic | Decision | Alternatives seen in the industry |
|---|---|---|
| Indentation | 3 spaces | 2 spaces (common), 4 spaces |
| Member / argument prefixes | none; use `this.` on collision, or a descriptive argument name where a linter forbids shadowing (`NAM-4`) | `m_` members (Doulos Easier UVM); `_arg` arguments |
| Class suffix | role in full (`bus_driver`), no `_c` | `_c` on every class (Advanced UVM / Hawkins style) |
| Instance names | short roles: `sqr drv mon cfg sb cov` | `m_sequencer`, `m_driver`, `m_config` |
| Constraint names | `<what>_cstr` | `c_<what>`, `<what>_c`, `<what>_cnstr` |
| Field macros | allowed on items and cfgs, banned in components (`FAC-3/4`) | banned entirely (Easier UVM); allowed everywhere |
| Configuration | cfg objects assigned top-down; `config_db` only for vifs and reuse boundaries (`CFG`) | `config_db` for every component (Easier UVM); explicit `configure()` calls |
| `p_sequencer` | banned; pass handles as members (`SEQ-8`) | `` `uvm_declare_p_sequencer `` (Easier UVM, Advanced UVM) |
| Virtual sequence sequencer | virtual sequencer if the env has one, else `null` | always a virtual sequencer |
| `super.<phase>()` | always in function phases; in task phases as the parent's contract says (`CMP-4`) | skip `super.build_phase` when extending UVM directly (Easier UVM) |
| `convert2string` vs `sprint` | `sprint` for dumps, `convert2string` for one-liners | `convert2string` only; `sprint` only |
| Message ID | functional static ID or `get_type_name()` | `get_name()` (rejected) |
| Agent analysis port | `ap = mon.ap` handle, no forwarding port | forwarded agent-level port; connect to `mon.ap` directly |
| Component phases | drivers, monitors, scoreboards use `run_phase` only | drivers override `reset_phase` |
| Randomize check | always check | check only when constraints can fail |
| Line length | 120 | 100, 150 |
| Coverage goals | no `option.weight` or `option.goal` (`COV-7`), except weight 0 on coverpoints that only build a cross | goals and weights per group to tune the total |
| Waivers | a `// <ID> waived: <reason>` comment at the code (§0) | PR description; separate lint waiver files |
| Formatter output | the formatter decides the layout it enforces (`FMT-13`) | hand formatting to a written standard |

## Acknowledgements

Several rules and rationales derive from the *Doulos Easier UVM Coding Guidelines* (© 2014–2016
Doulos) and from Brian Hunter, *Layering Protocols With Chained Sequencers* (Cavium) / *Advanced UVM*.
The remaining rules come from production verification experience.

## License

Copyright 2026 HEXK.

The text of this guide is licensed under the Creative Commons Attribution 4.0 International
License ([CC BY 4.0](LICENSE-docs)). Code examples in this guide are licensed under the
[Apache License 2.0](LICENSE) and may be copied into your own code, including proprietary code.
