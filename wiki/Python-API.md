# Python API

Besides the CLI, ACPYPE can be driven from Python. There are two entry points:

1. **`acpype.topol.ACTopol` / `MolTopol`** — the classes the CLI itself uses. Most
   flexible; writes files to disk.
2. **`acpype.acs_api.acpype_api(...)`** — a convenience wrapper that runs the full
   pipeline and returns all output files **as a JSON string of in-memory contents**
   (used by web-service / CCPN-style integrations).

---

## 1. `ACTopol` (ligand mode)

```python
from acpype.topol import ACTopol

mol = ACTopol(
    "ligand.mol2",
    chargeType="abcg2",   # gas | bcc | abcg2 | user
    atomType="gaff2",     # gaff | gaff2 | amber | amber2
    chargeVal=None,       # net charge; None -> guessed
    multiplicity="1",
    basename=None,
    outTopol="all",       # all | gmx | cns | charmm
    debug=False,
)

mol.createACTopol()       # antechamber -> tleap -> parmchk2 (AMBER topology)
mol.createMolTopol()      # parse + write GROMACS/CNS/CHARMM as requested
```

After `createMolTopol()`, the parsed model is available for inspection, e.g.:

```python
mol.molTopol.atoms[0].charge          # partial charge of the first atom
len(mol.molTopol.atoms)               # atom count
mol.molTopol.properDihedrals          # dihedral list
mol.chargeVal                         # net charge used
```

The constructor signature mirrors the CLI flags (`force`, `timeTol`, `qprog`, `ekFlag`,
`gmx4`, `merge`, `direct`, `is_sorted`, `chiral`, `verbose`, …); see
[`ACTopol.__init__`](../acpype/topol.py) and the
[Command-Line Reference](Command-Line-Reference) for meanings.

## 2. `MolTopol` (amb2gmx mode)

```python
from acpype.topol import MolTopol

mol = MolTopol(
    acFileXyz="system.inpcrd",
    acFileTop="system.prmtop",
    amb2gmx=True,
    basename="system",
)
mol.writeGromacsTopolFiles()
```

See [amb2gmx Mode](amb2gmx-Mode) for the caveats (solvent handling, `direct`).

## 3. `acpype_api` (files-in-memory, JSON out)

```python
import json
from acpype.acs_api import acpype_api

result = acpype_api("ligand.mol2", chargeType="abcg2", atomType="gaff2")
files = json.loads(result)

files["GMX_top"]      # GROMACS .top as a string
files["GMX_itp"]      # GROMACS .itp
files["AC_prmtop"]    # AMBER prmtop
files["mol2"]         # final MOL2 with charges
# ... CHARMM_*, CNS_*, GMX_OPLS_*, NEW_pdb, em_mdp, md_mdp, etc.
```

`acpype_api` runs the same `ACTopol` pipeline, reads each output file into memory, and
**removes the working folder** before returning — handy for services that don't want to
manage files on disk. On failure it returns JSON whose `file_name` carries the error
message (full traceback if `debug=True`).

> **Note:** `acpype_api` mirrors the CLI defaults and passes `chargeType` straight
> through (no allow-list), so `chargeType="abcg2"` works here exactly as on the CLI —
> provided your Antechamber is AmberTools ≥ 24.

---

## Tips & gotchas

- **Run this fork's code.** `import acpype` resolves to whatever is installed; with the
  upstream PyPI build, `abcg2` is unavailable. Use `pip install -e .` from the clone (or
  put the checkout on `PYTHONPATH`). See [Installation](Installation).
- **Working directory.** The pipeline `chdir`s into temporary/output folders; capture
  paths you need (`mol.absHomeDir`, `mol.rootDir`) before/after the calls.
- **CHARMM** still depends on `charmmgen` even via the API — see
  [Input & Output Formats](Input-and-Output-Formats).
- For interactive exploration from the CLI, `acpype -y` drops you into IPython with the
  `molecule` object in scope.
