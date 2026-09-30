# Optimizations, Architecture and Performance

The full list of optimization passes, levels and targets, how the two
optimizers stay language-agnostic, and benchmark figures.

## AST-Level Optimizations
- **Constant Folding**: Evaluate compile-time constant expressions
- **Constant Propagation**: Replace variables with their constant values
- **Algebraic Simplification**: x+0=x, x*1=x, x*0=0, etc.
- **Strength Reduction**: Replace expensive operations with cheaper equivalents
  - Multiply/divide by power-of-2 → shifts
  - Modulo by power-of-2 → bitwise AND
- **Dead Code Elimination**: Remove unreachable code
- **Common Subexpression Elimination (CSE)**: Eliminate redundant calculations
- **Copy Propagation**: Track variable aliases
- **Dead Store Elimination**: Remove unused assignments
- **Loop Optimizations**:
  - Loop-invariant code motion
  - Loop unrolling (configurable)
- **Boolean Simplification**: x=x→true, x<>x→false
- **Procedure Inlining**: Inline small procedures

## Peephole Optimizations
- **Pattern-based optimization** on Z80 assembly
- **Redundant load/store elimination**
- **Jump optimization** (including relative jumps for Z80)
- **Stack operation combining**
- **Register allocation cleanup**
- **8080 to Z80 mnemonic translation**

## Optimization Levels
- **Level 0**: No optimization
- **Level 1**: Basic (constant folding, algebraic simplification)
- **Level 2**: Standard (+ strength reduction, dead code elimination)
- **Level 3**: Aggressive (+ CSE, loop optimizations, inlining)

## Optimization Targets
- **Speed**: Optimize for execution speed
- **Size**: Optimize for code size
- **Balanced**: Balance between speed and size

## Architecture

upeep80 is designed to be language-agnostic:

### AST Optimizer
- Works on generic expression trees
- Requires minimal AST node interface:
  - Binary/unary expressions
  - Literals (integer, real)
  - Identifiers
  - Statements (assignment, if, loop, etc.)
- Can be adapted to any language's AST

### Peephole Optimizer
- Works directly on assembly text
- No knowledge of source language required
- Pattern-based transformation engine
- Configurable for 8080 or Z80 targets

## Performance

Benchmarks on typical compiler workloads:

- AST optimization: ~10,000 nodes/second
- Peephole optimization: ~50,000 instructions/second
- Memory usage: ~100MB for typical compilation unit

