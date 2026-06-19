# amb2gmx Mode (AMBER → GROMACS conversion)

ACPYPE's second mode reproduces and extends the classic **`amb2gmx.pl`** workflow: take
an **existing AMBER topology** (`prmtop` + `inpcrd`) and emit the equivalent GROMACS
files — no Antechamber, no charge assignment, no force-field choice. ACPYPE just parses
the AMBER topology objects and runs the format writers.

## Usage

```bash
acpype -p system.prmtop -x system.inpcrd
```

- `-p`/`--prmtop` and `-x`/`--inpcrd` must be given **together**.
- Do **not** combine with `-i` (ligand mode) — that's an error.
- `-b` sets the output basename/folder as usual.

This is triggered whenever `-i` is absent and `-p`/`-x` are present (see
[`cli.py`](../acpype/cli.py)); internally it constructs a `MolTopol` with `amb2gmx=True`
and calls `writeGromacsTopolFiles()`.

## Applicable options

Only options that affect *writing* apply here:

| Flag | Effect in amb2gmx |
|---|---|
| `-b` | output basename/folder |
| `-z` | GROMACS 4.0 RB dihedral style |
| `-g` | merge identical upper/lower-case atom types |
| `-l` | sort atoms for GROMACS ordering |
| `-j` | improper dihedrals for chiral atoms (CNS-style) |
| `-u` | **direct** conversion for any solvent (**experimental**) |
| `-d` / `-w` | debug / quiet |

Charge (`-c`), atom type (`-a`), net charge (`-n`), QM (`-q`/`-k`), and time limit
(`-s`) options are **not** meaningful here and are ignored — the charges and parameters
come straight from the input `prmtop`.

## Output

GROMACS topology/coordinate files plus the `em.mdp`/`md.mdp` run-parameter files, in
`<basename>.acpype/`. See [Input & Output Formats](Input-and-Output-Formats).

## Limitations

- **Solvent / multi-component systems.** The conversion is built around a molecule/
  system topology; when explicit **solvent** is present, the standard path may not
  yield a runnable `gmx mdrun` setup because solvent topology handling is incomplete.
  The `-u/--direct` flag attempts a direct conversion for any solvent but is explicitly
  **experimental** (the source notes that with solvent present `gmx mdrun` is not
  working due to missing solvent topology).
- `-u` is rejected outside amb2gmx mode.
- Non-uniform 1-4 scale factors (e.g. **GLYCAM06**) are handled (see the SoftwareX
  citation in [`acpype/__init__.py`](../acpype/__init__.py)), but verify the generated
  `[ pairs ]`/`[ defaults ]` blocks for such force fields.

## When to use which mode

- You have a **small molecule / ligand** and need parameters → ligand mode
  (`-i`), see [Quick Start](Quick-Start).
- You already produced an AMBER topology (e.g. via `tleap`, `LEaP`, parmed) and just
  want it in **GROMACS** → amb2gmx mode (`-p`/`-x`).
