# ABCG2 implementation — summary of changes

Changes made to add the `abcg2` (AM1-BCC-GAFF2) charge method to ACPYPE,
implemented agentically with [Claude Code](https://claude.com/claude-code)
(Anthropic, Claude Opus 4.8). For the session narrative see
[`CONVERSATION_LOG.md`](CONVERSATION_LOG.md).

## Empirical findings that shaped the implementation

- `abcg2` requires **`AmberTools >= 24`**. conda-forge's latest `ambertools` is
  **24.8** and supports it (charge method index 8 in `antechamber -L`); the bundled
  `AmberTools 21.11` does not (`Unknown charge method (abcg2).`).
- antechamber **defaults `abcg2` to `-eq 1`** (charge equalisation), same as `bcc`
  — so ACPYPE's existing generic `-c {chargeType}` command needs **no `-eq` change**.
- benzene reference charges under ABCG2: **C = −0.112, H = +0.112**.

## Files changed

### `acpype/parser_args.py`
- Added `"abcg2"` to the `-c/--charge_method` `choices`.
- Updated the help text to describe `abcg2` (AM1-BCC-GAFF2, requires
  AmberTools ≥ 24 and `-a gaff2`/`amber2`).

### `acpype/topol.py`
- `ACTopol.__init__`: emit a warning when `chargeType == "abcg2"` is combined with a
  non-GAFF2/amber2 atom type (i.e. `gaff`/`amber`), since ABCG2 is only validated
  against GAFF2 atom types. The default `-a` is already `gaff2`, so the common path
  is correct without changes.
- `execAntechamber`: runtime capability detection — when an `abcg2` run fails with
  `Unknown charge method` in the antechamber log, print an explicit error stating
  that `abcg2` requires AmberTools ≥ 24, instead of only dumping the raw log.
- `execAntechamber` docstring: updated the "List of the Charge Methods" table to
  include `ABCG2 abcg2 8` and a note about the AmberTools/GAFF2 requirement.
- No change to the command construction itself — `abcg2` flows through the existing
  generic `-c {chargeType}` plumbing, and `-eq` is left to antechamber's default.

### `acpype/__init__.py`
- Added the ABCG2 citation (He, X.; Man, V. H.; Gao, J.; Wang, J.
  *Development of an Improved Charge Model (ABCG2) for the GAFF2 Force Field.*
  J. Chem. Theory Comput. 2025, doi:10.1021/acs.jctc.4c01586) to the citations
  docstring.

### `tests/test_acpype.py`
- Added an `abcg2_required` skip marker (`pytest.mark.skipif`) that checks
  `antechamber -L` for `abcg2`, so the tests **skip gracefully** when only an older
  antechamber (e.g. the bundled AmberTools 21.11 in CI) is available.
- `test_abcg2`: runs `benzene.pdb` with `-c abcg2 -a gaff2`, asserts the antechamber
  command string and the benzene reference charges (C = −0.112, H = +0.112).
- `test_abcg2_atomtype_warning`: asserts the unvalidated-combination warning fires
  for `abcg2` + `-a gaff`.

### `README.md`
- Added a "Using ABCG2 Charges" subsection with a description, a worked example
  (`./run_acpype.py -i tests/benzene.pdb -c abcg2 -a gaff2`), and notes on the
  AmberTools ≥ 24 requirement and the GAFF2 atom-type pairing.
- Added the agentic-implementation attribution note linking to this `docs/abcg2/`
  directory.
- Trimmed the distribution-versions note to point at the new section.

## Verification

- Full test suite run against AmberTools 24.8: **45 passed** (including the 2 new
  abcg2 tests).
- With the bundled AmberTools 21.11, the abcg2 tests **skip** (verified) — existing
  CI stays green.
- The documented example command was run end-to-end and produced
  `benzene_abcg2_gaff2.mol2` plus the usual topology files.
- `ruff` reports no issues on the changed files.

## Not done (maintainer / distribution decisions)

- Rebuilding the bundled pip/Docker `antechamber`/`sqm` binaries to AmberTools ≥ 24
  (and shipping any new BCC parameter data file).
- Pinning a minimum `ambertools` version in the conda recipe.
- Adding an AmberTools ≥ 24 job to CI so `abcg2` is actively exercised there.

With a recent conda-forge `AmberTools`, these are optional: the feature degrades
gracefully (clear error message and skipped tests) when `abcg2` is unavailable.
