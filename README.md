# tiny-stack-vm-01 entry

A tiny stack VM for make.fun's "Build a tiny stack virtual machine" bounty
(tiny-stack-vm-01-run-20261122). It implements the kit's tiny stack ISA
exactly (`program/generator/subject/ISA.md`): 32-bit wrapping integers, a
256-value data stack, a 64-deep call stack, 256 words of memory, input /
output, all ten traps with their exit codes, and the step (1,000,000) and
output (65,536 bytes) limits.

## How it works

`main.ts` (Node 22, no dependencies) reads each instance under
`/input/<id>/` (`program.bin` + `input.bin`), runs the VM, and writes
`/output/<id>/stdout.bin` (exact bytes written) and `/output/<id>/exit.txt`
(exit code 0-255). Opcodes dispatch through a table of `[immediate bytes,
pops, pushes]`, so stack underflow (2) and overflow (3) are checked before
any effect, as the reference VM does. Arithmetic uses `Math.imul` (exact
low 32 bits); unsigned ops read operands with `>>> 0`; shifts mask the
count to 5 bits; `DIV`/`MOD` handle INT_MIN / -1. A trap ends the program
at once and keeps the output written before it.

## Run it

From the unpacked kit directory, on the practice set:

```
mkdir -p out && MAKEFUN_INPUT_DIR=practice/input MAKEFUN_OUTPUT_DIR=out node entry/main.ts
```

The grader runs `node /bot/main.ts` with `/input` and `/output` mounted.
One run answers all 40 instances in ~0.2s (limit 60s); two runs produce
byte-identical output (no clock, randomness, or environment reads).

## Limits

Correctness first: it matches the reference VM byte-for-byte on the whole
practice set (40/40, score 1,000,000) and on 3,000+ differential fuzz
cases plus targeted edge cases for every trap and tricky rule. It is a
plain interpreter, not a JIT — speed was not optimized beyond typed
arrays (`Int32Array`) for the stacks and memory.
