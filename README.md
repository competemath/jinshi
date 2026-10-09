# Jinshi

Jinshi examines the Tengoku tree for the ways a Lean 4 library can be wrong while every file still compiles: a kernel that was
bypassed, a statement that reads differently from what was checked, a theorem that says nothing, a hypothesis nobody needed, a
proof that is somebody else's. It is a soundness tester for a large, AI-heavy, auto-translated Lean corpus, and it is weird on
purpose. `docs/jinshi.md` is the full account: threat model, every examination, what Jinshi can and cannot promise.

## What is here

| path | what |
|---|---|
| `Jinshi/` | one Lean file per in-process examination (`tcb`, `shadow`, `nearname`, `arith`, `dossier`, `content`, `decide`, `duplicate`, `instdrift`, `unusedhyp`, `roundtrip`, `forensics`, `importance`, `lineage`, `necessity`, `entailed`, `arithUniverse`, `nested`, and the generator of `mutants`); `Base.lean` holds the shared `Finding` and context |
| `TengokuJinshi.lean` | the executable: imports every examination and registers it; regenerated from `Jinshi/` |
| `scripts/jinshi/run.py` | the round driver: kernel replay (leanchecker and lean4lean), autoImplicit re-elaboration, olean reproducibility, option audit, toolchain watch, then the executable's checks; emits one JSONL per check |
| `scripts/jinshi/partition.py` | the frozen 10-round partition of the tree (`tools/jinshi/partition.tsv`) |
| `scripts/jinshi/nanoda.py` | the third kernel's judge: Nanoda gets the same mutant modules and its verdict is compared with Lean's own |
| `scripts/jinshi/mutants.py` | the differential kernel fuzzer's judge: every generated mutant module is given to `leanchecker` and `lean4lean`, and a verdict that differs from Lean's own kernel is a `fail` with the reproducer |
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

Twenty-one examinations built and self-tested together: `jinshi selftest: 12 fixtures, 101 findings, 101 expected, 0 problems`.
The fuzzer's fixture gives 26 mutants, and Lean's kernel, `leanchecker` and `lean4lean` agree on every one. The sharded round
runs in the Tengoku sandbox; round 0 ran the driver's checks over 932 modules. `mutants`, `entailed` and `necessity` are costed
per theorem and run on the tree only when a round names them.

## License

Apache-2.0, like the rest of Tengoku.
