# ABCG2 implementation — Claude Code session log

ABCG2 (AM1-BCC-GAFF2) support was implemented **agentically with
[Claude Code](https://claude.com/claude-code)** (Anthropic, Claude Opus 4.8).
This file logs the conversation and the actions the agent took. A summary of the
resulting code/doc changes is in [`CHANGES.md`](CHANGES.md).

---

## 1. User request — implement ABCG2/GAFF2 support

The user supplied a detailed implementation assessment of what ABCG2 is and what
it would take to add it to ACPYPE, with the key points:

- ABCG2 ("AM1-BCC-GAFF2") is **not** a new charge-derivation algorithm — it is a
  new set of bond-charge-correction (BCC) parameters retuned against GAFF2, living
  entirely inside `antechamber`/`sqm` in AmberTools. ACPYPE only needs to invoke
  it (`-c abcg2`).
- The charge-method plumbing in ACPYPE is generic; the only allow-list is in
  `parser_args.py`. Output filenames already interpolate `chargeType`.
- Worth adding: an atom-type compatibility check (ABCG2 is validated against
  GAFF2), runtime capability detection (feature depends on AmberTools version,
  not ACPYPE version), and tests with real reference charges.
- The `-eq` (charge equalisation) default for `abcg2` needs **empirical**
  verification, not just doc-reading.

User notes added to the assessment:

- Recent `AmberTools` is available as a conda-forge install, so the bundled
  binaries probably do not need to be recompiled.
- For testing/verification, see <https://github.com/junmwang/abcg2> (molecules
  from the ABCG2 paper, <https://pmc.ncbi.nlm.nih.gov/articles/PMC11948320/>).

### Agent actions

1. **Inspected the source** — `parser_args.py`, `topol.py` (`ACTopol.__init__`,
   `guessCharge`, `execAntechamber`), `acs_api.py`, `params.py`,
   `tests/test_acpype.py`, `utils.py` (`set_for_pip`). Confirmed the charge plumbing
   is generic and `acs_api.py` does not restrict `chargeType`.
2. **Confirmed the blocker** — the bundled `antechamber`
   (`acpype/amber_linux`, AmberTools 21.11) has no `abcg2` charge method; running it
   emits `Unknown charge method (abcg2).`.
3. **Set up a capable toolchain** — created a conda env and installed
   `ambertools=24.8` from conda-forge. Observed that conda-forge's **latest is 24.8**
   (not 26 as the user expected), and that `antechamber -L` lists `ABCG2` as charge
   method index 8.
4. **Empirically verified the CLI behaviour** (the items flagged as needing
   verification):
   - `-c abcg2` flows through ACPYPE's existing generic command unchanged and
     produces `benzene_abcg2_gaff2.mol2`; antechamber reports `* Antechamber OK *`.
   - antechamber **defaults `abcg2` to `-eq 1`** (charge equalisation), the same as
     `bcc`. Verified by running `AAA.mol2` through `abcg2` with the default, with
     `-eq 0`, and with `-eq 1`, then diffing: default == `-eq 1` != `-eq 0`.
     **Conclusion: ACPYPE needs no explicit `-eq` flag.**
   - benzene reference charges under ABCG2: **C = −0.112, H = +0.112**
     (vs `bcc` −0.13 / +0.13).
5. **Made the code changes** (see `CHANGES.md`), **added capability-gated tests**,
   and ran the suite — **45 passed** (the 2 new abcg2 tests run when an
   abcg2-capable antechamber is present and skip otherwise).
6. Linted the changed files with `ruff` (clean).

## 2. User request — note the system already has Open Babel

The user pointed out that the `obabel` binary (Open Babel 3.1.1) is installed
system-wide.

### Agent action

Clarified that the binary alone is not sufficient: ACPYPE's chiral-group detection
imports the **python `openbabel` module** (`from openbabel import openbabel`/`pybel`),
which was not present in the fresh conda env — so `openbabel-wheel` was pip-installed
into the test env. This is purely a local verification detail, not a code change.

## 3. User request — document the changes in the README and add an ABCG2 example

### Agent action

Added a **"Using ABCG2 Charges"** subsection to `README.md` in the usage flow, with:

- a short description of ABCG2 and the He et al. 2025 citation,
- a working example: `./run_acpype.py -i tests/benzene.pdb -c abcg2 -a gaff2`
  (verified to run end-to-end and produce `benzene_abcg2_gaff2.mol2`),
- notes on the `AmberTools >= 24` requirement and the GAFF2 atom-type pairing.

Trimmed the earlier one-line note in the distribution-versions block to a pointer
to the new section to avoid duplication.

## 4. User request — state that this was implemented agentically with Claude Code, and include a log of the conversation and changes (as separate files)

### Agent action

Added an attribution note to the README's ABCG2 section linking to two dedicated
files under `docs/abcg2/`:

- [`CONVERSATION_LOG.md`](CONVERSATION_LOG.md) — this file.
- [`CHANGES.md`](CHANGES.md) — summary of the changes made.
