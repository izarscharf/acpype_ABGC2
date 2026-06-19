# ABCG2 Charge Method

**ABCG2** ("AM1-BCC-GAFF2") is the feature that distinguishes this fork. It is a new
**bond-charge-correction (BCC) parameter set** layered on the *same* AM1-BCC machinery
already used for standard `bcc` charges, but re-tuned specifically against **GAFF2** to
improve hydration free energies.

> He, X.; Man, V. H.; Gao, J.; Wang, J. *Development of an Improved Charge Model
> (ABCG2) for the GAFF2 Force Field.* J. Chem. Theory Comput. **2025**.
> doi:[10.1021/acs.jctc.4c01586](https://doi.org/10.1021/acs.jctc.4c01586) ·
> [PMC11948320](https://pmc.ncbi.nlm.nih.gov/articles/PMC11948320/) ·
> reference molecules: <https://github.com/junmwang/abcg2>

Because ABCG2 lives entirely inside `antechamber`/`sqm`, ACPYPE's job is only to invoke
it correctly. The charge derivation, atom typing and AM1 pre-charge all happen in
AmberTools.

## Usage

```bash
acpype -i ligand.pdb -c abcg2 -a gaff2
```

Output charges land in `<base>_abcg2_gaff2.mol2` and propagate to all topology files.

## Requirements

- **AmberTools ≥ 24.** `abcg2` (charge-method index 8 in `antechamber -L`) was
  introduced in recent AmberTools; conda-forge's `ambertools` (24.8) supports it. The
  **bundled AmberTools 21.11** (pip/Docker fallback) does **not**.
- **GAFF2 atom types.** Use `-a gaff2` (default) or `-a amber2`.

Check availability:

```bash
antechamber -L | grep -i abcg2     # expect: ABCG2  abcg2  8
```

## How this fork implements it

The Python footprint is intentionally tiny, because the charge plumbing is generic:

- [`parser_args.py`](../acpype/parser_args.py) — `abcg2` added to the `-c` choices.
- [`topol.py`](../acpype/topol.py):
  - `execAntechamber` already builds `-c <method>` generically, so `abcg2` flows
    through unchanged — **no command special-casing**.
  - A **warning** fires if `abcg2` is paired with a non-GAFF2/amber2 atom type.
  - A **capability check**: if an `abcg2` run fails with *"Unknown charge method"*,
    ACPYPE reports a clear "requires AmberTools ≥ 24" error instead of a raw log dump.
- [`__init__.py`](../acpype/__init__.py) — the He et al. (2025) citation.

Implementation details, the empirical verification, and the change list are recorded in
[`docs/abcg2/`](../docs/abcg2/) (`CONVERSATION_LOG.md`, `CHANGES.md`).

## Empirically verified behaviour

These were confirmed against AmberTools 24.8 (they can't be settled by reading docs):

- **`-eq` (charge equalisation):** antechamber defaults `abcg2` to `-eq 1`, the same as
  `bcc`. So ACPYPE relies on antechamber's default and passes **no explicit `-eq`** —
  verified by diffing output for default vs `-eq 0` vs `-eq 1`.
- **Reference charges (benzene):** ABCG2 gives **C = −0.112, H = +0.112**
  (vs `bcc` −0.13 / +0.13) — the symmetric, expected result, and the value asserted in
  the test suite.

## Limitations & caveats

- Needs a recent AmberTools; not usable from the default bundled binaries or the current
  Docker image.
- Only validated for GAFF2 — combining with `-a gaff`/`amber` is out-of-domain (warned).
- Like `bcc`, depends on a successful AM1 calculation and on the input geometry; large or
  strained molecules may need a higher `-s` or tuned `-k` QM keywords.
- The fork does **not** rebuild the bundled pip/Docker binaries or pin the conda recipe
  to AmberTools ≥ 24 — providing a modern AmberTools (e.g. via
  [`environment.yml`](../environment.yml)) is the supported route.

## See also

- [Charge Methods](Charge-Methods) — comparison with `gas`/`bcc`/`user`.
- [Atom Types & Force Fields](Atom-Types-and-Force-Fields) — why GAFF2 is required.
- [Installation](Installation) — getting an AmberTools ≥ 24 toolchain.
