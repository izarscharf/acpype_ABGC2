# Input & Output Formats

## Input formats (`-i`)

| Input | Extension / form | Requires OpenBabel? | Notes |
|---|---|---|---|
| PDB | `.pdb` | **yes** | Converted to MOL2 internally; needs sensible elements/connectivity. |
| Sybyl MOL2 | `.mol2` | no | Preferred: carries atom names, bonds and (for `-c user`) charges. |
| MDL Molfile | `.mdl` or `.mol` | no | Treated as MDL (`mdl`) by Antechamber. |
| SMILES | a quoted string, e.g. `'CCCC'` | **yes** | Converted to 3D MOL2 via OpenBabel (`pybel`). Output folder defaults to `smiles_molecule.acpype` unless `-b` is given. |

**OpenBabel requirement:** PDB and SMILES inputs use the **OpenBabel Python module**
(`pybel`), which must be installed in the *same* Python environment as ACPYPE. The
`obabel` binary alone is not sufficient. MOL2/MDL inputs do not need OpenBabel.

> The net charge is guessed from the input (Gasteiger pass) when `-n` is not given; for
> charged species pass `-n` explicitly to avoid charge-drift errors.

---

## Output formats (`-o`)

Output is written into `<basename>.acpype/`. `-o all` (default) writes everything;
`-o gmx|cns|charmm` restricts the engine-specific topologies. **AMBER files are always
produced** because they are the source the others are converted from.

### AMBER (always)

| File | Contents |
|---|---|
| `<base>_AC.prmtop` | topology + parameters |
| `<base>_AC.inpcrd` | coordinates |
| `<base>_AC.lib` | residue library |
| `<base>_AC.frcmod` | modified/estimated parameters (from `parmchk2`) |
| `<base>_<charge>_<atomtype>.mol2` | final MOL2 with assigned charges/types |
| `<base>_NEW.pdb` | cleaned PDB produced by ACPYPE |

### GROMACS (`-o gmx`)

| File | Contents |
|---|---|
| `<base>_GMX.top` | full topology |
| `<base>_GMX.itp` | molecule `[ moleculetype ]` include |
| `<base>_GMX.gro` | coordinates |
| `posre_<base>.itp` | position restraints |
| `em.mdp`, `md.mdp` | ready-to-edit energy-minimisation / MD run parameters |
| `<base>_GMX_OPLS.top` / `.itp` | **experimental** OPLS/AA mapping |

Flags affecting GROMACS output: `-z` (GMX 4.0 RB dihedrals), `-g` (merge identical
upper/lower-case atom types), `-l` (sort atoms).

### CNS/XPLOR (`-o cns`)

| File | Contents |
|---|---|
| `<base>_CNS.top` | topology |
| `<base>_CNS.par` | parameters |
| `<base>_CNS.inp` | run parameters |

Flags: `-t` (allhdg-like parameters, experimental), `-j` (improper dihedrals for chiral
atoms).

### CHARMM (`-o charmm`)

| File | Contents |
|---|---|
| `<base>_CHARMM.rtf` | topology |
| `<base>_CHARMM.prm` | parameters |
| `<base>_CHARMM.inp` | run parameters |

> ⚠️ **CHARMM output requires the `charmmgen` binary.** It ships only with the
> **bundled** AmberTools binaries (a `charmmgen` from AmberTools17). The conda-forge
> `ambertools` package does **not** include it, so **CHARMM topologies are not
> generated** in conda or plain `git` installs — only via the pip/Docker bundle.
> This is an upstream packaging fact, independent of this fork's changes.

---

## The `.acpype` output folder

- One folder per run, named from the input or `-b`.
- ACPYPE **caches**: if the folder already exists it is reused; pass `-f` to force a
  clean recomputation after changing inputs/options.
- Temporary working files are removed on success unless `-d/--debug` is set.
