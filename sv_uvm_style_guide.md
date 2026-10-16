# HEXK SystemVerilog / UVM Style Guide

Version 0.2 · 2026-10-16 · Scope: verification code (testbench, UVM, VIP). RTL is out of scope.

Copyright 2026 HEXK. Text: [CC BY 4.0](LICENSE-docs). Code examples: [Apache-2.0](LICENSE).

This guide is written for two readers: verification engineers and AI coding assistants.
Every rule has a stable ID (for example `NAM-4`) so that reviews, lint waivers and AI
prompts can cite it. Rules are short; the reason follows in *italics* only when it is not obvious.

---

## 0. How to read and apply this guide

**Keywords.** **MUST** = required; a review blocks on it. **SHOULD** = default; deviate only with a
reason (a comment or the PR description). **MAY** = allowed, not required.

**Precedence.**

1. The SystemVerilog LRM (IEEE 1800) and the UVM standard (IEEE 1800.2) always win.
2. When **creating** a new file or directory, follow this guide.
3. When **editing** an existing file that has a clear, consistent local style, match the file
   for formatting and naming. Do not restyle lines you are not otherwise changing.
   Correctness rules (`LNG`, `SEQ`, `OBJ`, `CHK`, `MSG` severity) still apply to new code.
4. If this guide is silent, copy the structure of the 2–3 nearest similar files.

**Instructions for AI assistants.**

- Apply rules by ID. When you deviate, say which rule and why in your reply, not in the code.
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
| FMT-2 | **MUST** use ASCII only; no trailing whitespace; files end with a single newline. |
| FMT-3 | **SHOULD** keep lines ≤ 120 characters. Break long argument lists one argument per line. |
| FMT-4 | **MUST** use `begin`/`end` for every `if`, `else`, `for`, `foreach`, `repeat`, `while`, `forever` body and `case` item that contains more than one statement. **SHOULD** use them even for a single statement. *Prevents dangling-`else` and "added a second line" bugs.* |
| FMT-5 | `begin` goes on the same line as its keyword; `end` lines up with the line that opened it; `end else begin` stays on one line. |
| FMT-6 | **MUST** close every `class`, `function`, `task`, `module`, `interface`, `package`, `covergroup`, `property`, `sequence`, `clocking` with an end label: `endclass : bus_driver`. *The compiler checks labels; it does not check comments.* |
| FMT-7 | **MUST NOT** put end comments on `end` (`end // while (...)`). Turn off editor auto-insertion of them. |
| FMT-8 | One declaration or statement per line. |
| FMT-9 | One blank line between methods; never two or more consecutive blank lines. |
| FMT-10 | **SHOULD** align consecutive declarations and assignments in columns when it aids scanning (types, names, `=`). Do not realign untouched code. |
| FMT-11 | One space after keywords and around binary operators: `if (a == b)`, `foreach (q[i])`. No space between a function name and `(`. Parameter lists: `uvm_driver #(bus_item)`. |
| FMT-12 | **SHOULD** keep files under ~500 lines. A larger file usually means a class doing two jobs. |

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
| NAM-1 | **MUST** use `lower_snake_case` for classes, variables, functions, tasks, instances, signals, files, packages, interfaces, modules. No `camelCase`. *Matches the UVM library.* |
| NAM-2 | **MUST** use `UPPER_SNAKE_CASE` for `parameter`, `localparam`, `` `define `` constants and enum literals. |
| NAM-3 | **MUST NOT** prefix anything with `uvm_` or `m_`. *`uvm_` belongs to the library; `m_` marks UVM internals (`m_sequencer`), and mixing in our own names hides that distinction.* |
| NAM-4 | **No member or argument prefixes.** Class members and method arguments use plain names. When an argument shadows a member, disambiguate with `this.`. Where the project's linter treats such shadowing as an error (for example slang `-Wshadow-value`), name the argument for what it is to the member instead (`new_cfg`, `uid_parent`, `item_name`). **MUST NOT** use a prefix or suffix that only marks an argument (`a_`, `i_`, `_arg`). |
| NAM-5 | **SHOULD** spell words out. Abbreviate only with the approved short forms in §2.3. `config_base_seq`, not `cnfg_base_seq`. |
| NAM-6 | **SHOULD** use names that state purpose: `first_stage_timer`, not `timer1`. Short names (`i`, `idx`, `item`) are fine for small local scopes. |
| NAM-7 | **MUST** restrict UVM instance-name strings to `[a-zA-Z0-9_]`, with no double underscores. |
| NAM-8 | **MUST** pass a `create()` name string identical to the variable name: `drv = bus_driver::type_id::create("drv", this);`. In loops use `$sformatf("agent_%0d", i)`. *Log paths then map directly to code.* |
| NAM-9 | Active-low signals end in `_n` (`rst_n`). Testbench-only nets in a module or interface **SHOULD** start with `tb_` to separate them from DUT signals. |

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

`addr cfg cnt cov cp cstr data drv env idx if intf* mon pkg req rsp sb seq sqr vif vseq vsqr`
(*`intf` only where an existing codebase already uses it; new code uses `_if`.)

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
| FIL-7 | **MUST NOT** use `` `timescale ``. Declare `timeunit`/`timeprecision` in the testbench top module and in interfaces. **MUST** give every delay literal a unit: `#10ns`, `#(period_ps * 1ps)`. Never a bare `#10`. |
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
| DOC-2 | Comment **why**, not what: intent, spec references (`// Spec 4.2.1: ...`), protocol corner cases, non-obvious trade-offs, waivers. |
| DOC-3 | **MUST NOT** write comments that restate a name or the code (`// Constructor`, `// build phase`, `// Class: foo`), banner boxes around standard methods, or template leftovers (`// Insert code here`). |
| DOC-4 | **SHOULD** use `//` line comments. `/* */` is acceptable only for the file header. |
| DOC-5 | Document public API members (knobs, ports, methods others call) with one line above the declaration. |
| DOC-6 | **MUST** format TODOs as `// TODO:<owner>:<P1\|P2\|P3>: <text>`, for example `// TODO:rg:P1: handle concurrent interrupts`. **MUST** resolve P1s before sign-off. *Greppable and summarizable.* |
| DOC-7 | **MUST** delete dead and commented-out code before merging. Version control remembers it. |
| DOC-8 | Units of physical or time quantities go in the name or the comment: `timeout_ns`, `freq_hz`. |

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
| CLS-1 | **MUST** order class contents as follows: (1) public API members (knobs, ports, handles, `rand` fields); (2) internal members, marked `local` or `protected`; (3) `` `uvm_*_utils `` registration; (4) constraints; (5) covergroups; (6) `new`; (7) UVM phase methods in phase order; (8) other public methods; (9) `local`/`protected` helper methods. |
| CLS-2 | **MUST** define methods inside the class body. **MUST NOT** use `extern` prototypes. *Maintaining two copies of every signature costs more than it gives; editors outline classes.* |
| CLS-3 | **SHOULD** mark internal state `local` (or `protected` when extensions need it). |
| CLS-4 | **MUST NOT** repeat `virtual` when overriding a UVM virtual method (`build_phase`, `do_copy`). **MUST** declare new methods that extensions may override `virtual`. |
| CLS-5 | **SHOULD** keep methods short. Split long phase bodies into named helpers (`build_agents()`, `connect_scoreboard()`). |
| CLS-6 | **SHOULD** declare variables in the narrowest scope. Do not promote a task's local to a class member. |
| CLS-7 | **SHOULD NOT** initialize variables to their default value (`int count = 0;`). |
| CLS-8 | Constructors of UVM classes: components `function new(string name, uvm_component parent)`; objects `function new(string name = "<class_name>")`. The body is `super.new(...)` plus, only if needed, covergroup construction or analysis-port construction. Classes that do not derive from a UVM class (plain utility classes, classes of a UVM-free package) take the arguments they need. |

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
| LNG-15 | Use `local::` for outer-scope variables inside `randomize() with {}`. |
| LNG-16 | Performance: prefer `inside` and set operators over `foreach` in constraints; do no arithmetic inside constraint loops; prefer masks to `%`; split large constraint sets into staged randomization; randomize object arrays element by element. |
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
| LNG-20 | When a failed `$cast` would be a bug, call it as a **task** (ignore the return value). The simulator reports and stops. |
| LNG-21 | When failure is an expected outcome (type dispatch), call it as a **function** inside `if` or `case (1)`. |

```systemverilog
function void write(uvm_sequence_item t);
   bus_item    bus_t;
   irq_item    irq_t;
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
| CMP-4 | **MUST** call `super.<phase>(phase)` first in every phase method you override. *Uniform, and safe for extension.* |
| CMP-5 | Drivers, monitors, scoreboards, coverage and models **MUST** run in `run_phase` only and be phase-agnostic. They handle reset by watching the reset signal or a reset event, not by overriding `reset_phase`. *Lets them be reused under any test phasing.* |
| CMP-6 | Tests (and envs, for default stimulus) **MAY** use `reset_phase`, `configure_phase`, `main_phase` and `shutdown_phase` to start sequences. **SHOULD NOT** override `pre_*`/`post_*` run-time phases; keep them as cross-domain sync points. |
| CMP-7 | **SHOULD NOT** use phase jumps. If one is unavoidable, document the cleanup each component performs. |
| CMP-8 | An agent contains only the canonical `sqr`, `drv`, `mon` (plus protocol-intrinsic checks and coverage). It builds `sqr` and `drv` only when `get_is_active() == UVM_ACTIVE`; it always builds `mon`. |
| CMP-9 | Agents are protocol-specific and DUT-agnostic. DUT-specific checking and coverage live outside the agent (`CHK`, `COV`). |
| CMP-10 | An agent exposes its monitor's analysis port as an agent-level handle (`ap = mon.ap;` in `connect_phase`). It does not add a second, forwarded port. *Users can connect at agent level without hidden plumbing.* |
| CMP-11 | Envs **MUST** work both as the top env and as a sub-env (`REU-1`). |
| CMP-12 | Monitors **MUST NOT** drive interface signals and **MUST NOT** block on anything except the interface. |
| CMP-13 | Drivers pull items just-in-time (`get_next_item` → drive → `item_done`). Use `try_next_item` when the protocol needs idle cycles driven between items. |
| CMP-14 | Variable delays within or between items live in the item (`delay_cycles`), so sequences control timing. Only drivers, monitors and interfaces consume time for the protocol; other components (scoreboards, predictors, coverage) do not delay items. Supervision and timing utilities (watchdogs, timeouts, end-of-test polling, delay models whose purpose is time) **MAY** wait. |

---

## 8. Factory and macros (`FAC`)

| ID | Rule |
|---|---|
| FAC-1 | **MUST** create every component, sequence, sequence item and config object through the factory: `type_id::create("name"[, this])`. Never call `new` directly, except for TLM ports, covergroups and `uvm_event`s. |
| FAC-2 | **MUST** register every class derived from `uvm_object` (components included) with `` `uvm_component_utils `` / `` `uvm_object_utils `` (or the `_param_` variants). Interface classes, plain utility classes and classes of a UVM-free package are not registered. |
| FAC-3 | Field macros (`` `uvm_field_* ``) are **allowed on sequence items and config objects**, for print, compare, copy and record. Flag metadata `UVM_NOCOMPARE` and pick a radix (`UVM_HEX`) where it helps. |
| FAC-4 | **MUST NOT** use field macros in components. *They trigger implicit config-db lookups (`apply_config_settings`). Configuration is explicit here (`CFG`).* |
| FAC-5 | When field macros are not used, implement `do_copy`, `do_compare`, `do_print`, and `do_record` if transactions are recorded. Implement `do_pack`/`do_unpack` when the item crosses DPI or a packed boundary. |
| FAC-6 | On hot paths (a high-rate copy or compare per transaction), hand-write `do_copy`/`do_compare` instead of relying on field macros, and measure. |
| FAC-7 | **MUST NOT** use the `` `uvm_do* ``, `` `uvm_create* ``, `` `uvm_send* `` or `` `uvm_rand_send* `` macro families. Write `create` → `start_item` → `randomize` → `finish_item` explicitly. |
| FAC-8 | **MUST NOT** use `` `uvm_declare_p_sequencer ``. *It locks a sequence to one sequencer type.* See `SEQ-8` for alternatives. |
| FAC-9 | **MUST** use the report macros (`` `uvm_info `` and friends), never `uvm_report_*()` calls directly. *The macros skip message construction when the message is filtered.* |
| FAC-10 | Overrides: `orig::type_id::set_type_override(new_t::get_type())` or `set_inst_override`. Use `uvm_factory::get()` to reach the factory. |
| FAC-11 | **MUST NOT** use deprecated UVM features or undocumented UVM internals (members starting with `m_`, except `m_sequencer`, which is documented). |

---

## 9. Configuration (`CFG`)

| ID | Rule |
|---|---|
| CFG-1 | Each configurable component has **one config object** (`<owner>_cfg extends uvm_object`). The config object holds `is_active`, the virtual interface, enables (`has_coverage`, `has_checks`) and protocol knobs. |
| CFG-2 | Configuration flows **top-down by direct assignment**. The parent creates or receives the child's cfg and sets `child.cfg = ...` in its own `build_phase`, after `create` and before the child's `build_phase` runs. |
| CFG-3 | Parent config objects **contain** their children's config objects and construct them in `new`. One `randomize()` of the top cfg then randomizes everything, and cross-cfg constraints are possible. |
| CFG-4 | `uvm_config_db` is used only (a) to hand virtual interfaces from the testbench module to the test, and (b) across a reuse boundary where the parent cannot see the child type. Every `get` **MUST** be checked and **MUST** use `get(this, "", "<field>", ...)`. |
| CFG-5 | **MUST NOT** use `uvm_resource_db` directly. **MUST NOT** call `uvm_config_db` in run-time loops. *It is slow, and it costs most at integration level.* Exception: the `REG::` resources that UVM's register sequences read (`NO_REG_TESTS`, `NO_REG_BIT_BASH_TEST`, ...) are set and read through `uvm_resource_db`, as UVM defines them. |
| CFG-6 | **MUST** validate that a mandatory cfg or vif was received early (`build_phase`/`connect_phase`) with `` `uvm_fatal ``. Never let a null handle crash later. |
| CFG-7 | Agents override `get_is_active()` to return `cfg.is_active`. |
| CFG-8 | Env cfgs carry `has_<agent>` bits (default 1) that gate building the agent, fetching its vif, and connecting it (`REU-2`). |
| CFG-9 | Settings that change at run time and status shared between components live in config objects (handles), never re-fetched from `uvm_config_db`. |
| CFG-10 | Tests customize through a `virtual function void modify_cfg()` hook called before cfg randomization in the base test, not by rewriting `build_phase`. |

```systemverilog
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
| IFC-1 | One SystemVerilog interface instance per DUT interface. The UVM side reaches it only through a virtual interface handle. |
| IFC-2 | **MUST** generate clocks and resets in a module (the testbench top or a clock/reset agent's interface), never in classes and never in a `program`. **MUST NOT** use `program` blocks. |
| IFC-3 | **SHOULD** use clocking blocks for synchronous signals: `drv_cb` (outputs driven, inputs sampled) and `mon_cb` (inputs only). Add modports `drv_mp`/`mon_mp` that expose the clocking blocks plus any asynchronous signals. |
| IFC-4 | Drivers access only `drv_cb`/`drv_mp`; monitors access only `mon_cb`/`mon_mp`. |
| IFC-5 | Hierarchical references into the DUT (probes, internal coverage) **MUST** be confined to one bound interface or module. Classes never contain hierarchical paths. |
| IFC-6 | Low-level protocol assertions (`assert property`) and temporal coverage (`cover property`) live in the interface. Label every assertion; the failure message goes through `$error`/`` `uvm_error ``. |
| IFC-7 | Non-class code reports with `$info`/`$warning`/`$error`/`$fatal`, never `$display`. |
| IFC-8 | **SHOULD**, for emulation/acceleration readiness, put pin-level driving and sampling in interface tasks (BFM style). Keep driver and monitor classes thin and untimed except for calls into the interface. |
| IFC-9 | The testbench top `<block>_tb` instantiates `dut`, the interfaces, the clock/reset generation and the binds. It sets the vifs into `uvm_config_db` and calls `run_test()`. It contains no other test logic. |

```systemverilog
interface bus_if (input logic clk, input logic rst_n);
   timeunit 1ns;
   timeprecision 1ps;

   logic        valid;
   logic        ready;
   logic        write;
   logic [31:0] addr;
   logic [31:0] wdata;
   logic [31:0] rdata;

   clocking drv_cb @(posedge clk);
      output valid, write, addr, wdata;
      input  ready, rdata;
   endclocking : drv_cb

   clocking mon_cb @(posedge clk);
      input valid, ready, write, addr, wdata, rdata;
   endclocking : mon_cb

   modport drv_mp (clocking drv_cb, input rst_n);
   modport mon_mp (clocking mon_cb, input rst_n);

   valid_stable_a: assert property (@(posedge clk) disable iff (!rst_n)
      valid && !ready |=> valid && $stable(addr))
      else $error("valid dropped or addr changed before ready");
endinterface : bus_if
```

---

## 11. Transactions (`TRN`)

| ID | Rule |
|---|---|
| TRN-1 | Extend `uvm_sequence_item`, never `uvm_transaction`. |
| TRN-2 | **SHOULD** use one item class per agent for both driver and monitor. Response fields (read data, status) are non-`rand` members of the same item. |
| TRN-3 | Fields that could be randomized are `rand`. Metadata (timestamps, IDs, debug) is not `rand` and is flagged `UVM_NOCOMPARE`. |
| TRN-4 | Constraints in an item define **legality** only. Scenario shaping belongs in sequences. |
| TRN-5 | **SHOULD** provide a short one-line `convert2string()` for log messages. Full dumps use `sprint()` (`MSG-7`). |

```systemverilog
class bus_item extends uvm_sequence_item;
   rand bus_op_e                 op;
   rand bit [ADDR_WIDTH-1:0]     addr;
   rand bit [DATA_WIDTH-1:0]     data;
   rand int unsigned             delay_cycles;
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
| SEQ-1 | Reusable stimulus lives in sequences and virtual sequences. Tests select, configure and start them. Test-only code that will never be reused stays in the test. **MUST NOT** create a sequence that only one test will ever use. |
| SEQ-2 | Generate items with the explicit pattern: `create` → `start_item` → `randomize` (late randomization, after `start_item`) → `finish_item`. |
| SEQ-3 | **MUST** `randomize()` every sequence object before `start()`, even if it has no `rand` members. *A factory override may add some.* |
| SEQ-4 | Leaf sequences are always started on a real sequencer, with the parent sequence passed when nested: `seq.start(sqr, this)`. *Needed for locks, grabs, priorities, interrupt handling and sequencer-scoped config.* |
| SEQ-5 | Virtual sequences are started on the env's virtual sequencer if it has one, otherwise on `null`. They receive sub-sequencer handles as members set before `start()`. |
| SEQ-6 | Register accesses inside sequences **MUST** pass `.parent(this)`. |
| SEQ-7 | Sequences are not phase-aware. They **MUST** run correctly when started from any run-time phase. |
| SEQ-8 | A sequence that needs data beyond its sequencer gets it from (a) members set by the starter, or (b) its cfg object set before start. **MUST NOT** use `p_sequencer` (`FAC-8`). |
| SEQ-9 | Use control knobs: `rand` members of the sequence (`num_items`, `burst_len`) that `body()` uses and the starter constrains. |
| SEQ-10 | Housekeeping (objections, locks) goes in `pre_start`/`post_start`, never `pre_body`/`post_body`. `body()` holds only behaviour. |
| SEQ-11 | **MUST NOT** use `default_sequence`. **MAY** use `uvm_sequence_library` for random selection among sequences. |
| SEQ-12 | Directed bug-reproduction sequences are named after the bug: `bus_bug1234_seq`. |
| SEQ-13 | Use `uvm_event`/`uvm_barrier` handles, passed via cfg, for cross-sequence synchronization. Do not poll signals from sequences. |
| SEQ-14 | Override `pre_do`/`mid_do`/`post_do` only to change a sequence you cannot or should not edit, together with a factory override. |
| SEQ-15 | Layered protocols use one sequencer per layer, each with its own item type. Translation (layering) sequences on each sequencer move items between layers; drivers and monitors stay at the lowest layer and stay thin. |

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
| OBJ-1 | The test raises **one** objection around its top-level stimulus in each run-time phase it uses and drops it when the stimulus completes. |
| OBJ-2 | Completion of in-flight traffic is **event-driven**: a scoreboard (or watchdog) holds an objection while it has outstanding expected items and drops it when they drain. **MUST NOT** wait with fixed delays (`#10us`) at the end of a test. |
| OBJ-3 | **SHOULD NOT** raise or drop objections per transaction in hot paths. If a component must, it raises once when its queue goes non-empty and drops once when it empties. |
| OBJ-4 | **MUST** pass a description string to every `raise_objection`/`drop_objection`. *Shown by `+UVM_OBJECTION_TRACE`.* |
| OBJ-5 | **SHOULD** call `set_propagate_mode(0)` on phase objections in the base test. *Hierarchical propagation costs simulation time and is rarely needed.* |
| OBJ-6 | Sequences that object do so in `pre_start`/`post_start`, guarded by `get_starting_phase() != null`, and drop in `do_kill` if they can be killed. |
| OBJ-7 | Use a global timeout (`uvm_root::set_timeout` or a watchdog component) as a safety net, not as the end-of-test mechanism. |

---

## 14. TLM and analysis connections (`TLM`)

| ID | Rule |
|---|---|
| TLM-1 | Components communicate through TLM ports, exports and imps, not through direct handles to each other. Exceptions: a test or virtual sequence referencing an agent's sequencer; a parent setting `child.cfg`. |
| TLM-2 | Use analysis ports for monitor → subscriber traffic (one-to-many, may be unconnected). |
| TLM-3 | Connect port → export directly. When buffering is needed, put the `uvm_tlm_analysis_fifo` **inside** the consuming component. |
| TLM-4 | A component with several inputs uses `` `uvm_analysis_imp_decl(_<what>) `` (declared at package scope) with descriptive suffixes (`_expected`, `_actual`), not generic `_imp1`. |
| TLM-5 | Analysis `write()` implementations **MUST NOT** consume time and **MUST NOT** modify the received object. Clone first if you need to store and mutate it. |

---

## 15. Checking, scoreboards, models (`CHK`)

| ID | Rule |
|---|---|
| CHK-1 | Put each check where it is simplest: signal-level protocol checks in SVA (`IFC-6`); end-to-end data checks in scoreboards and models; simple read-after-write checks in the sequence or test. |
| CHK-2 | Scoreboards and models live in the env, outside agents, and are fed by analysis connections. |
| CHK-3 | For simple in-order flows use `uvm_in_order_comparator` (or an equivalent shared component) instead of a hand-rolled one. |
| CHK-4 | Store expected items in structures matched to the lookup: queues for in-order, associative arrays keyed by ID or address for out-of-order. **MUST NOT** do linear searches over large queues per transaction. |
| CHK-5 | **MUST** report leftovers in `check_phase`: expected items never matched, and actual items never expected. |
| CHK-6 | Scoreboards **MUST** flush state on reset and drop any objection they hold. |
| CHK-7 | Interrupt handling uses a dedicated handler component. It locks the relevant sequencers, runs an ISR sequence, then unlocks. *This only works if every sequence runs on a sequencer (`SEQ-4`).* |

---

## 16. Messaging (`MSG`)

| ID | Rule |
|---|---|
| MSG-1 | UVM classes report only with `` `uvm_info ``/`` `uvm_warning ``/`` `uvm_error ``/`` `uvm_fatal `` (or the `_context` variants). Classes of a UVM-free package report with the severity tasks `$info`/`$warning`/`$error`/`$fatal`. **MUST NOT** use `$display` in classes. |
| MSG-2 | Message ID: a static, functional ID (`"AXI_RD_RSP"`, `"SB_MISMATCH"`), or `get_type_name()` when nothing more specific fits. **MUST NOT** use `get_name()`. *Instance names depend on where the class is instantiated and are already printed in the message header.* |
| MSG-3 | Verbosity: `UVM_NONE`, almost never; `UVM_LOW`, rare test-level milestones; `UVM_MEDIUM`, the default for transaction-level activity; `UVM_HIGH`/`UVM_DEBUG`, testbench internals and tracing. |
| MSG-4 | Severity: `UVM_INFO`, information; `UVM_WARNING`, suspicious but possibly legal; `UVM_ERROR`, a definite failure where simulation can usefully continue; `UVM_FATAL`, simulation cannot meaningfully continue (missing cfg, randomize failure, broken environment). |
| MSG-5 | Remember that envs are reused vertically. A message per packet at `UVM_MEDIUM` from every block floods full-chip logs. **SHOULD** print whole packets at `UVM_HIGH` or above. |
| MSG-6 | **MUST** guard expensive message construction (loops, large `sprint()`) with `if (uvm_report_enabled(UVM_HIGH, UVM_INFO, "<ID>")) begin ... end`. **MUST NOT** guard with `get_report_verbosity_level()`, which ignores ID-level settings. |
| MSG-7 | Print objects with `sprint()` so the active printer policy controls the format. `convert2string()` is for one-line summaries only (`TRN-5`). |
| MSG-8 | Messages say what was expected, what was observed, and where: `"rdata mismatch addr=0x%0h exp=0x%0h act=0x%0h"`. |
| MSG-9 | A passing test produces **no** `UVM_ERROR` or `UVM_FATAL`. Expected errors (negative tests) are demoted or caught with `set_report_severity_id_override` or a report catcher, keyed by message ID. |

```systemverilog
if (uvm_report_enabled(UVM_HIGH, UVM_INFO, "SB_QUEUE")) begin
   foreach (expected_q[i]) begin
      `uvm_info("SB_QUEUE", $sformatf("expected_q[%0d]: %s", i, expected_q[i].convert2string()), UVM_HIGH)
   end
end
```

---

## 17. Functional coverage (`COV`)

| ID | Rule |
|---|---|
| COV-1 | Covergroups live in coverage components (`uvm_subscriber` or a component with `` `uvm_analysis_imp_decl `` inputs). They are constructed in `new()`. Never put covergroups in items, sequences or monitors. |
| COV-2 | Sample explicitly with `cg.sample()` (or a `with function sample(...)` covergroup), not with a clock event. |
| COV-3 | Sample what the DUT did (outputs, internal state via a bound interface, register values changed by the DUT), not the stimulus that was sent. |
| COV-4 | Keep specification coverage and implementation coverage in separate covergroups. |
| COV-5 | Gate coverage collection with `cfg.has_coverage`. |
| COV-6 | Exclude values with `ignore_bins`. **MUST NOT** use `illegal_bins`; illegal values are errors that assertions or checkers must report. |
| COV-7 | **SHOULD** set `option.at_least` above 1 where repeated hits matter. **MUST NOT** set `option.weight` or `option.goal`. Exception: weight 0 on coverpoints that exist only to build a cross, so that the cross alone counts. |
| COV-8 | Design bins from the verification plan: typical, boundary and special values. Keep `iff` guards simple. |
| COV-9 | Use `cover property` in interfaces for temporal protocol coverage. |

---

## 18. Register layer (`REG`)

| ID | Rule |
|---|---|
| REG-1 | Generate the register model; never hand-edit it (`FIL-10`). |
| REG-2 | Each env that uses registers has a `regmodel` handle. The env builds the model only when `regmodel == null` (top level); otherwise the parent assigns the sub-block. |
| REG-3 | The register hierarchy mirrors the DUT hierarchy. Child blocks are named after their agent or env. |
| REG-4 | Use explicit prediction: a `uvm_reg_predictor` connected to the bus monitor, with `set_auto_predict(0)`. |
| REG-5 | Each bus agent that carries register traffic provides a `<bus>_reg_adapter`, setting `provides_responses` when the driver returns responses. |
| REG-6 | Register sequences extend `uvm_sequence` (or a shared register base sequence), have a `regmodel` member set before `start()`, and pass `.parent(this)` on every access. |
| REG-7 | Check the returned `uvm_status_e` of every register access. |

---

## 19. Tests and pass/fail (`TST`)

| ID | Rule |
|---|---|
| TST-1 | Test hierarchy, at most four levels: `uvm_test` → project/common base test (reporting, printer, timeout, objection policy) → `<block>_base_test` (builds the env and cfg, runs reset and init) → `<block>_<feature>_test`. Prefer composing sequences over deep test inheritance. |
| TST-2 | The base test exposes hooks: `virtual function void do_type_overrides()`, `virtual function void modify_cfg()`. Concrete tests override hooks, not `build_phase`. |
| TST-3 | The env **MUST** run legal default traffic under an empty test, or the base test **MUST** provide a smoke test. |
| TST-4 | Select tests with `+UVM_TESTNAME` and `run_test()` with no argument. Prefer plusargs to recompiling for knobs. Exceptions: a top that must also run without UVM's command-line DPI (`UVM_NO_DPI`) **MAY** read `+UVM_TESTNAME` itself and pass it to `run_test()`; a self-contained example **MAY** pass its own test name. |
| TST-5 | Tests **MUST NOT** reach deep into the hierarchy with long hard-coded paths. Use env-provided handles or accessors. |
| TST-6 | Pass criterion = **no errors** **and** a positive check that the expected work happened (data compared, scoreboard drained with a non-zero match count). "No errors" alone does not count as a pass. |
| TST-7 | **MUST NOT** pass or fail on message or packet counts alone. Prove stimulus reached the DUT through coverage. |
| TST-8 | Wrap third-party VIP behind a thin vendor-neutral adapter (cfg, item, sequences). Vendor-specific code stays inside the wrapper. |

---

## 20. Reuse and integration (`REU`)

| ID | Rule |
|---|---|
| REU-1 | Design every env for vertical reuse from day one: no assumptions that it is the top, no global state, all external dependencies passed in through cfg. |
| REU-2 | Integration de-duplicates agents through cfg `has_<agent>` flags. To keep consumers fed when an agent is turned off, the env routes every agent stream through an env-level `uvm_analysis_export`: agent → env export → consumers. The env also offers env-level analysis ports for external connection. The integrating env then connects the single remaining agent to every env export. |
| REU-3 | Create env-level exports only when at least one consumer exists (gate on the consumer's `has_*` flag). *An export with no imp behind it is a connection error.* |
| REU-4 | Adapt reused components with factory overrides and cfg, never by editing or copying their source. |
| REU-5 | Chip-specific packages extend or parameterize generic VIP. They **MUST NOT** fork it. |
| REU-6 | Parameterized agents **SHOULD** offer a `typedef` per concrete use: `typedef bus_agent #(.DATA_WIDTH(64)) bus64_agent_t;`. |

---

## Appendix A — Complete minimal agent

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
      repeat (item.delay_cycles) begin
         @(vif.drv_cb);
      end
      vif.drv_cb.valid <= 1'b1;
      vif.drv_cb.write <= (item.op == BUS_WRITE);
      vif.drv_cb.addr  <= item.addr;
      vif.drv_cb.wdata <= item.data;
      do begin
         @(vif.drv_cb);
      end while (vif.drv_cb.ready !== 1'b1);
      if (item.op == BUS_READ) begin
         item.data = vif.drv_cb.rdata;
      end
      item.status = UVM_IS_OK;
      drive_idle();
   endtask : drive_item
endclass : bus_driver
```

```systemverilog
// bus_monitor.sv
// Publishes every completed bus transfer on ap. Passive; never drives the interface.
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
      bus_item item;
      forever begin
         @(vif.mon_cb iff (vif.mon_cb.valid === 1'b1 && vif.mon_cb.ready === 1'b1));
         item      = bus_item::type_id::create("item");
         item.op   = vif.mon_cb.write ? BUS_WRITE : BUS_READ;
         item.addr = vif.mon_cb.addr;
         item.data = vif.mon_cb.write ? vif.mon_cb.wdata : vif.mon_cb.rdata;
         ap.write(item);
      end
   endtask : run_phase
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
// bus_smoke_test.sv
// Smoke test: 20 random transfers, then waits for the scoreboard to drain.
class bus_smoke_test extends bus_base_test;
   `uvm_component_utils(bus_smoke_test)

   function new(string name, uvm_component parent);
      super.new(name, parent);
   endfunction : new

   task main_phase(uvm_phase phase);
      bus_random_seq seq = bus_random_seq::type_id::create("seq");
      super.main_phase(phase);
      phase.raise_objection(this, "bus_smoke_test stimulus");
      if (!seq.randomize() with { num_items == 20; }) begin
         `uvm_fatal("RAND", "bus_random_seq randomization failed")
      end
      seq.start(env.bus_agent.sqr);
      phase.drop_objection(this, "bus_smoke_test stimulus done");
   endtask : main_phase
endclass : bus_smoke_test
```

---

## Appendix B — Review checklist

Run this on every change, whether you are a human reviewer or an AI before handing back code.

- [ ] 3-space indent, no tabs, ≤ 120 columns, `begin`/`end` everywhere, end labels on every block (`FMT`)
- [ ] Names: `lower_snake`, constants `UPPER`, no `m_`/`uvm_` prefixes, suffixes per §2.2, `create()` string = variable name (`NAM`)
- [ ] File header present and meaningful; no restating comments; no dead code; TODOs formatted (`DOC`)
- [ ] Every class is in a package; no `extern`; no compilation-unit imports; no `` `timescale ``; all delays carry units (`FIL`, `CLS`)
- [ ] Every `randomize()` is checked with `` `uvm_fatal ``; no `assert(randomize())`; constraints are named `_cstr` (`LNG`)
- [ ] Factory used for every object; no `` `uvm_do* ``; no `p_sequencer`; no field macros in components (`FAC`)
- [ ] cfg handed down by the parent; mandatory cfg/vif checked with `` `uvm_fatal `` early; no `uvm_config_db` in loops (`CFG`)
- [ ] Sequences started on a sequencer with parent `this`; sequences randomized before `start()`; `.parent(this)` on register accesses (`SEQ`, `REG`)
- [ ] Drivers, monitors and scoreboards use `run_phase` only; no fixed end-of-test delays; objections carry descriptions (`CMP`, `OBJ`)
- [ ] Message IDs are meaningful (never `get_name()`); verbosity set deliberately; expensive prints guarded (`MSG`)
- [ ] Scoreboard reports leftovers in `check_phase`; passing requires positive evidence of work done (`CHK`, `TST`)
- [ ] Zero new compile or elaboration warnings (`LNG-32`)

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
| `super.<phase>()` | always call (`CMP-4`) | skip `super.build_phase` when extending UVM directly (Easier UVM) |
| `convert2string` vs `sprint` | `sprint` for dumps, `convert2string` for one-liners | `convert2string` only; `sprint` only |
| Message ID | functional static ID or `get_type_name()` | `get_name()` (rejected) |
| Agent analysis port | `ap = mon.ap` handle, no forwarding port | forwarded agent-level port; connect to `mon.ap` directly |
| Component phases | drivers, monitors, scoreboards use `run_phase` only | drivers override `reset_phase` |
| Randomize check | always check | check only when constraints can fail |
| Line length | 120 | 100, 150 |

## Acknowledgements

Several rules and rationales derive from the *Doulos Easier UVM Coding Guidelines* (© 2014–2016
Doulos) and from Brian Hunter, *Layering Protocols With Chained Sequencers* (Cavium) / *Advanced UVM*.
The remaining rules come from production verification experience.

## License

Copyright 2026 HEXK.

The text of this guide is licensed under the Creative Commons Attribution 4.0 International
License ([CC BY 4.0](LICENSE-docs)). Code examples in this guide are licensed under the
[Apache License 2.0](LICENSE) and may be copied into your own code, including proprietary code.
