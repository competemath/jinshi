# Jinshi

Jinshi examines the Tengoku tree for the ways a Lean 4 library can be wrong while every file still compiles: a kernel that was
bypassed, a statement that reads differently from what was checked, a theorem that says nothing, a hypothesis nobody needed, a
proof that is somebody else's. It is a soundness tester for a large, AI-heavy, auto-translated Lean corpus, and it is weird on
purpose. `docs/jinshi.md` is the full account: threat model, every examination, what Jinshi can and cannot promise.

## What is here

| path | what |
|---|---|
| `Jinshi/` | one Lean file per in-process examination (`tcb`, `shadow`, `nearname`, `arith`, `dossier`, `content`, `decide`, `duplicate`, `instdrift`, `unusedhyp`, `roundtrip`, `forensics`, `importance`, `lineage`, `necessity`, `entailed`); `Base.lean` holds the shared `Finding` and context |
| `TengokuJinshi.lean` | the executable: imports every examination and registers it; regenerated from `Jinshi/` |
| `scripts/jinshi/run.py` | the round driver: kernel replay (leanchecker and lean4lean), autoImplicit re-elaboration, olean reproducibility, option audit, toolchain watch, then the executable's checks; emits one JSONL per check |
| `scripts/jinshi/partition.py` | the frozen 10-round partition of the tree (`tools/jinshi/partition.tsv`) |
| `scripts/jinshi/merge.py`, `report.py` | merge the shards of a round; read a round's findings per library, check and severity |
| `scripts/jinshi/selftest.py` | runs every examination on `tools/jinshi/fixtures/*.lean` and checks each against its `.expected.tsv`; the forged fixture must be refused by both kernels |
| `.github/workflows/jinshi.yml` | one round in shards: plan, a matrix of runners, merge |
| `lakefile.jinshi.toml` | the `lean_lib` and `lean_exe` entries the host lakefile needs |

## How it is used

Jinshi lives on top of a Tengoku checkout: copy these paths into the tree at the same locations, add the entries of
`lakefile.jinshi.toml` to the tree's `lakefile.toml`, then

```bash
lake build tengoku-jinshi
JINSHI_LEAN4LEAN=/path/to/lean4lean python3 scripts/jinshi/selftest.py
python3 scripts/jinshi/run.py --round 0 --out jinshi-out
python3 scripts/jinshi/report.py jinshi-out
```

Findings are JSON lines `{check, severity, module, name, line, detail}` with severity `fail`, `warn` or `info`. A `fail` means
the kernel's verdict and the tree disagree; `warn` is something a maintainer reads; `info` is the record.

## Status

Eighteen examinations built and self-tested on their own, plus `importance` and `lineage` whose fixtures do not yet pass (the importance indexer counts zero uses where the fixture plants two: the first thing to fix). The merge of `necessity` and `entailed` into this tree has not yet been built and self-tested together. The sharded round runs in the Tengoku sandbox. In progress: `mutants`
mutated proofs judged by three independent kernels, a disagreement being a kernel bug), `entailed` (the tree proves a new
theorem from what it already had, with a checked certificate), `necessity` (every hypothesis justified by a counterexample).
