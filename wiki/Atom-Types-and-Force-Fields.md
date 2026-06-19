# Atom Types & Force Fields (`-a`)

`-a/--atom_type` selects how atoms are typed and therefore which force-field
parameters are assigned. It also picks the `tleap` source file and the `gaff*.dat`
parameter file used internally.

| `-a` value | Atom typing | Bonded/vdW parameters | `tleap`/`.dat` used | Typical use |
|---|---|---|---|---|
| `gaff` | GAFF | General Amber Force Field (original) | `leaprc.gaff`, `gaff.dat` | legacy GAFF ligands |
| `gaff2` *(default)* | GAFF2 | GAFF2 (improved) | `leaprc.gaff2`, `gaff2.dat` | modern small-molecule MD |
| `amber` | AMBER (ff14SB) | protein ff14SB types | `leaprc.protein.ff14SB` | residues/fragments in AMBER protein FF |
| `amber2` | AMBER + GAFF2 | ff14SB + GAFF2 | ff14SB + `gaff2` | mixed standard-residue + general atoms |

Internally the rule is simply: if the atom-type string contains a **`2`** (i.e.
`gaff2`/`amber2`), ACPYPE uses the GAFF2 parameter set (`leaprc.gaff2` / `gaff2.dat`);
otherwise the original GAFF set.

---

## GAFF vs GAFF2

- **GAFF** is the original General Amber Force Field for organic molecules.
- **GAFF2** is its successor with revised parameters and is the recommended default for
  new work. ACPYPE defaults to `gaff2`.

## `amber` / `amber2`

These use the **ff14SB** (protein) leap source. They are intended for standard
biomolecular residues or fragments that should carry AMBER protein-style atom types,
optionally combined with GAFF2 (`amber2`) for any non-standard atoms.

- **Limitation:** ACPYPE parameterises a **single molecule/residue unit**; it is not a
  protein builder. The `amber`/`amber2` options control atom typing for that unit, not
  whole-protein assembly (use `tleap`/`pdb2gmx` directly for full proteins).

---

## Interaction with charge methods

- **`abcg2` is only validated against GAFF2 atom types.** Use `-a gaff2` (default) or
  `-a amber2`. Combining `abcg2` with `-a gaff`/`-a amber` produces charges outside the
  model's validated domain and ACPYPE will warn. See
  [Charge Methods](Charge-Methods) and [ABCG2](ABCG2-Charge-Method).
- All other charge methods (`gas`/`bcc`/`user`) work with any atom type.

## Output naming

The atom type appears in the final MOL2 name, e.g. `ligand_abcg2_gaff2.mol2`,
`ligand_bcc_gaff.mol2`. The GROMACS/AMBER/CNS/CHARMM files are named by basename and
engine (see [Input & Output Formats](Input-and-Output-Formats)).

## Missing parameters

When Antechamber/`tleap` cannot find a parameter, ACPYPE runs **`parmchk2`** to produce
an `frcmod` of estimated/penalised parameters. Inspect `<base>_AC.frcmod` — large
penalty scores or `ATTN` markers indicate parameters you should review or refit before
trusting the topology.
