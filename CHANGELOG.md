# Changelog

All notable changes to upeep80, the peephole optimizer for 8080/Z80 compilers,
are documented here.

## [0.2.1] - 2026-08-20

### Added
- A `LICENSE` file, containing the GNU General Public License version 3. The
  repository previously shipped no license text at all. Note that the README
  and the `license` field in `pyproject.toml` both describe the project as
  GPL v2, so the text now in the tree and the packaging metadata disagree;
  the metadata is what PyPI displays.
- `.github/workflows/publish.yml`, which builds an sdist and wheel and uploads
  them to PyPI through a trusted publisher when a GitHub release is published
  or the workflow is dispatched by hand. No effect on the installed package.

### Changed
- The README now opens with a notice that the project is no longer maintained,
  and says which optimizer to use instead: `upeepz80` for compilers emitting
  pure lowercase Z80 mnemonics (`ld`, `jp`, `jr`), upeep80 for compilers
  emitting 8080 mnemonics (`MOV`, `MVI`, `LXI`) that still need translation to
  Z80, which `upeepz80` does not do.
- The GitHub URLs in `pyproject.toml` and the README are real. All four
  `[project.urls]` entries — Homepage, Repository, Documentation and Bug
  Tracker — pointed at `github.com/yourusername/upeep80`, the unedited
  placeholder, so every project link on the PyPI page for 0.2.0 was a 404, as
  were the `git clone` line and the uplm80/uada80 links in the README.
- The README no longer advertises documentation that does not exist. It listed
  `docs/API.md`, `docs/OPTIMIZATION_GUIDE.md`, `docs/CUSTOM_PATTERNS.md`,
  `CONTRIBUTING.md` and an `examples/` directory with three named example
  scripts; none of them are in the repository, and none ever were. The "See
  Also" section of `docs/INTEGRATION.md` listed the same three dead links and
  has been removed outright. `docs/INTEGRATION.md` is the only guide there is.

### Fixed
- Peephole optimizer: 8080 `JP` (jump if plus) is now translated to Z80
  `JP P,`. `_translate_instruction` in `upeep80/peephole.py` had no branch for
  `JP` at all, so the instruction was returned unchanged — and on the Z80,
  `JP addr` is an unconditional jump. Every 8080-syntax `JP` passed through
  `optimize_8080(..., target=Target.Z80)` came out with its condition silently
  discarded and the branch taken every time. There was no diagnostic, because
  the emitted line is a perfectly valid Z80 instruction, just not the one the
  input asked for. Comparison code is hit directly: the optimizer's own 16-bit
  compare patterns `sbb_mov_ora_jp` and `sub_16bit_sign_jp` are sign tests that
  end in `JP`. Operands containing a comma are left alone, so already
  conditional forms are not touched twice, as are `JP (HL)` and the literal
  targets `JP 0` and `JP 5` (CP/M warm boot and the BDOS entry, written
  Z80-style). Those two literals and `(HL)` are the entire escape list: an
  unconditional Z80 `JP label` appearing in input that was declared to be 8080
  syntax now becomes `JP P,label`. That is a behavior change and not only a
  fix: mixed-syntax input that relied on the old pass-through will now branch
  conditionally. The escape list is matched case-sensitively, so `jp (hl)` in
  lowercase input comes out as `JP P,(hl)`.
- Peephole optimizer: the accumulator normalization in `_translate_instruction`
  no longer mangles the Z80 16-bit adds. `ADD` and `ADC` were rewritten to
  `ADD A,<operands>` / `ADC A,<operands>` whenever the operands did not already
  begin with `A,`, so a 16-bit add reaching the translator picked up a third
  operand: `ADD IX,SP` became `ADD A,IX,SP`, and `ADD HL,DE`, `ADD IY,BC` and
  `ADC HL,DE` were corrupted the same way. `ADD IX,SP` is the frame-pointer
  setup in a reentrant procedure prologue, so that prologue came out as
  something that is not a Z80 instruction: the build fails at the assembler, or,
  on an M80-compatible assembler that drops surplus operands, quietly assembles
  as `ADD A`. `ADD` now excludes the `HL,`, `IX,` and `IY,` prefixes and `ADC`
  excludes `HL,`, matching the Z80 instruction set. The prefix test is
  case-sensitive and the operands are not upper-cased before it runs, so
  lowercase input is still mangled: `add hl,de` still comes out as
  `ADD A,hl,de`.

Neither fix has a regression test. The repository has no `tests/` directory, in
spite of `pyproject.toml` pointing pytest at one, so nothing in this release is
covered by an automated test.

History before 0.2.1 was never written down; it is in the git log.
