# ⚠️ THIS PROJECT IS NO LONGER MAINTAINED ⚠️

If your compiler emits **pure lowercase Z80** mnemonics (`ld`, `jp`, `jr`), use
[upeepz80](https://github.com/avwohl/upeepz80) instead — it is the maintained
optimizer for that input.

upeep80 remains here for compilers that emit **8080** mnemonics (`MOV`, `MVI`,
`LXI`) needing translation to Z80; upeepz80 does not handle that case.

---

# upeep80 - Universal Peephole Optimizer for 8080/Z80 (ARCHIVED)

A language-agnostic optimization library for 8080 and Z80 compilers.

## Overview

upeep80 provides high-quality optimization passes for compilers targeting the Intel 8080 and Zilog Z80 processors. The library is designed to be reusable across different source languages.

## Features

- **AST-level optimizations**: constant folding and propagation, algebraic
  simplification, strength reduction, dead code and dead store elimination,
  CSE, loop optimizations and procedure inlining
- **Peephole optimizations** on 8080/Z80 assembly text, including 8080 to Z80
  mnemonic translation
- **Optimization levels** 0-3, and speed, size or balanced targets

[docs/OPTIMIZATIONS.md](https://github.com/avwohl/upeep80/blob/main/docs/OPTIMIZATIONS.md) lists every pass.

## Installation

```bash
pip install upeep80
```

Or for development:

```bash
git clone https://github.com/avwohl/upeep80.git
cd upeep80
pip install -e ".[dev]"
```

## Usage

### AST Optimization

```python
from upeep80 import ASTOptimizer, OptimizeFor

# Create optimizer
optimizer = ASTOptimizer(
    opt_level=2,
    optimize_for=OptimizeFor.BALANCED
)

# Optimize your AST
optimized_ast = optimizer.optimize(ast_tree)

# Check statistics
print(f"Constants folded: {optimizer.stats.constants_folded}")
print(f"Dead code eliminated: {optimizer.stats.dead_code_eliminated}")
```

### Peephole Optimization

```python
from upeep80 import PeepholeOptimizer, Target

# Create optimizer
optimizer = PeepholeOptimizer(
    target=Target.Z80,
    opt_level=2
)

# Optimize assembly code
optimized_asm = optimizer.optimize(assembly_lines)

# Check statistics
print(f"Patterns applied: {optimizer.stats.patterns_applied}")
print(f"Instructions eliminated: {optimizer.stats.instructions_eliminated}")
```

## Used By

- **[uplm80](https://github.com/avwohl/uplm80)** - PL/M-80 compiler for Z80
- **[uada80](https://github.com/avwohl/uada80)** - Ada compiler for Z80

## Documentation

- [Integration Guide](https://github.com/avwohl/upeep80/blob/main/docs/INTEGRATION.md)
- [docs/OPTIMIZATIONS.md](https://github.com/avwohl/upeep80/blob/main/docs/OPTIMIZATIONS.md) - every optimization pass, the optimization levels and targets, the architecture, and performance figures
- [docs/DEVELOPMENT.md](https://github.com/avwohl/upeep80/blob/main/docs/DEVELOPMENT.md) - running tests, type checking, code formatting
- [CHANGELOG.md](https://github.com/avwohl/upeep80/blob/main/CHANGELOG.md) - what changed in each version

## Contributing

Contributions are welcome. Open an issue or a pull request.

## License

This project is licensed under the GNU General Public License v2.0 - see [LICENSE](https://github.com/avwohl/upeep80/blob/main/LICENSE) for details.

## History

upeep80 was extracted from the [uplm80](https://github.com/avwohl/uplm80) project to provide a reusable optimization library for multiple retro compiler projects targeting the 8080/Z80 architecture.

## Acknowledgments

- Optimization techniques based on classic compiler optimization literature
- Pattern-based peephole optimization inspired by Davidson & Fraser's work
- Z80 instruction set reference from [z80.info](http://z80.info)

## See Also

- [uplm80](https://github.com/avwohl/uplm80) - PL/M-80 compiler
- [uada80](https://github.com/avwohl/uada80) - Ada compiler for Z80
- [Z80 CPU User Manual](http://www.z80.info/zip/z80cpu_um.pdf)
