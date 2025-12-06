# Integration Guide for upeep80

## Overview

upeep80 provides two main optimization components:
1. **AST Optimizer** - Language-agnostic high-level optimizations
2. **Peephole Optimizer** - Assembly-level pattern optimizations

## Peephole Optimizer Integration

The peephole optimizer is completely language-agnostic and works directly on assembly text.

### Basic Usage

```python
from upeep80 import PeepholeOptimizer, Target

# Create optimizer
optimizer = PeepholeOptimizer(
    target=Target.Z80,  # or Target.I8080
    opt_level=2
)

# Optimize assembly code (list of strings or single string)
optimized = optimizer.optimize(assembly_lines)
```

### No Integration Required

The peephole optimizer requires no adaptation - just pass it your assembly output.

## AST Optimizer Integration

The AST optimizer requires your AST nodes to implement a specific interface.

### Required AST Node Structure

Your AST must have nodes that match these patterns. The optimizer uses duck typing, so exact class names don't matter - only the structure.

#### Expression Nodes

```python
# All expressions should have a 'span' attribute (optional)
class Expr:
    span: Optional[SourceSpan] = None

# Literals
class NumberLiteral(Expr):
    value: int

class StringLiteral(Expr):
    value: str

# Identifiers
class Identifier(Expr):
    name: str

# Binary expressions
class BinaryExpr(Expr):
    op: BinaryOp  # Enum with ADD, SUB, MUL, DIV, etc.
    left: Expr
    right: Expr

# Unary expressions
class UnaryExpr(Expr):
    op: UnaryOp  # Enum with NEG, NOT, ABS, etc.
    operand: Expr

# Other expression types:
# - SubscriptExpr (array indexing)
# - MemberExpr (struct/record member access)
# - CallExpr (function calls)
# - LocationExpr (address-of)
# - EmbeddedAssignExpr (assignment within expression)
```

#### Statement Nodes

```python
# Statements
class AssignStmt(Stmt):
    targets: list[Expr]
    value: Expr

class IfStmt(Stmt):
    condition: Expr
    then_stmt: Stmt
    else_stmt: Optional[Stmt]

class DoBlock(Stmt):  # Block/compound statement
    decls: list[Decl]
    stmts: list[Stmt]
    end_label: Optional[str]

# Loop statements
class DoWhileBlock(Stmt):
    condition: Expr
    stmts: list[Stmt]

class DoIterBlock(Stmt):  # For loop
    index_var: Expr
    start: Expr
    bound: Expr
    step: Optional[Expr]
    stmts: list[Stmt]

# Other statement types:
# - CallStmt, ReturnStmt, GotoStmt, HaltStmt
# - NullStmt, LabeledStmt
# - DoCaseBlock (switch/case)
```

#### Declaration Nodes

```python
class VarDecl(Decl):
    names: list[str]
    data_type: DataType
    initial_values: Optional[list[Expr]]

class ProcDecl(Decl):
    name: str
    params: list[ParamDecl]
    return_type: Optional[DataType]
    is_external: bool
    is_reentrant: bool
    interrupt_num: Optional[int]
    decls: list[Decl]
    stmts: list[Stmt]
```

### Using the AST Optimizer

#### Option 1: Adapt Your AST (Recommended)

Create wrapper classes that implement the required interface:

```python
from upeep80 import ASTOptimizer, OptimizeFor

# Your AST nodes
class MyExpr:
    pass

class MyBinaryExpr(MyExpr):
    def __init__(self, operator, left, right):
        self.op = operator  # Convert to upeep80.BinaryOp
        self.left = left
        self.right = right

# Use optimizer
optimizer = ASTOptimizer(opt_level=2, optimize_for=OptimizeFor.BALANCED)
optimized_ast = optimizer.optimize(my_module)
```

#### Option 2: Copy and Adapt

Since the AST optimizer is tightly coupled to AST structure, you can:

1. Copy `ast_optimizer.py` to your project
2. Modify it to work with your specific AST nodes
3. Keep the optimization logic intact

The optimization algorithms are the valuable part - the AST interface can be adapted.

### Operator Enums

Your AST should define these operator enums (or map to them):

```python
from enum import Enum, auto

class BinaryOp(Enum):
    # Arithmetic
    ADD = auto()
    SUB = auto()
    MUL = auto()
    DIV = auto()
    MOD = auto()

    # Logical
    AND = auto()
    OR = auto()
    XOR = auto()

    # Relational
    EQ = auto()
    NE = auto()
    LT = auto()
    LE = auto()
    GT = auto()
    GE = auto()

class UnaryOp(Enum):
    NEG = auto()
    NOT = auto()
    ABS = auto()
    LOW = auto()   # Low byte
    HIGH = auto()  # High byte
```

## Example: Complete Integration

### For PL/M Compiler (uplm80)

```python
from upeep80 import ASTOptimizer, PeepholeOptimizer, Target, OptimizeFor

# 1. Parse source code
ast = parse_plm_source(source_code)

# 2. Optimize AST
ast_optimizer = ASTOptimizer(opt_level=2, optimize_for=OptimizeFor.SIZE)
optimized_ast = ast_optimizer.optimize(ast)

print(f"Constants folded: {ast_optimizer.stats.constants_folded}")
print(f"Dead code eliminated: {ast_optimizer.stats.dead_code_eliminated}")

# 3. Generate assembly
assembly = generate_code(optimized_ast)

# 4. Optimize assembly
peephole = PeepholeOptimizer(target=Target.Z80, opt_level=2)
final_assembly = peephole.optimize(assembly)

print(f"Patterns applied: {peephole.stats.patterns_applied}")
```

### For Ada Compiler (uada80)

```python
from upeep80 import ASTOptimizer, PeepholeOptimizer, Target, OptimizeFor

# 1. Parse Ada source
compilation_unit = parse_ada_source(ada_code)

# 2. Semantic analysis
analyze_semantics(compilation_unit)

# 3. Optimize AST
# Note: May need adapter layer for Ada AST
optimizer = ASTOptimizer(opt_level=3, optimize_for=OptimizeFor.BALANCED)
optimized_unit = optimizer.optimize(compilation_unit)

# 4. Generate Z80 assembly
assembly = generate_z80_code(optimized_unit)

# 5. Peephole optimization
peephole = PeepholeOptimizer(target=Target.Z80, opt_level=3)
final_assembly = peephole.optimize(assembly)
```

## Optimization Statistics

Both optimizers track statistics:

```python
# AST Optimizer stats
print(f"Constants folded: {optimizer.stats.constants_folded}")
print(f"Strength reductions: {optimizer.stats.strength_reductions}")
print(f"Dead code eliminated: {optimizer.stats.dead_code_eliminated}")
print(f"CSE eliminations: {optimizer.stats.cse_eliminations}")
print(f"Loop invariants moved: {optimizer.stats.loop_invariants_moved}")
print(f"Procedures inlined: {optimizer.stats.procedures_inlined}")

# Peephole stats
print(f"Patterns applied: {peephole.stats.patterns_applied}")
print(f"Instructions eliminated: {peephole.stats.instructions_eliminated}")
```

## Custom Peephole Patterns

You can add custom patterns:

```python
from upeep80 import PeepholeOptimizer, PeepholePattern

# Define custom pattern
custom_pattern = PeepholePattern(
    name="eliminate_redundant_load",
    pattern=[
        ("LD", "A,*"),
        ("LD", "A,*"),
    ],
    replacement=[
        ("LD", "A,{1}"),  # Keep second load only
    ]
)

optimizer = PeepholeOptimizer(target=Target.Z80)
optimizer.add_pattern(custom_pattern)
```

## Notes

- The AST optimizer is more language-specific and may require adaptation
- The peephole optimizer is completely language-agnostic
- Both components are independent - use either or both
- Optimization levels 0-3 control aggressiveness
- OptimizeFor controls size vs. speed tradeoffs

## See Also

- [API Reference](API.md)
- [Optimization Guide](OPTIMIZATION_GUIDE.md)
- [Custom Patterns](CUSTOM_PATTERNS.md)
