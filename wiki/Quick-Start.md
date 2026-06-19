# Quick Start

These assume you have a working install with `antechamber`/`tleap`/`parmchk2` (and
`obabel` for PDB/SMILES) on your `PATH` — see [Installation](Installation). Replace
`acpype` with `./run_acpype.py` if you are running from the repo without installing.

Every run creates a folder **`<basename>.acpype/`** in the current directory holding
all output files. See [Input & Output Formats](Input-and-Output-Formats) for what's
inside.

---

## The most common runs

```bash
# A ligand from PDB, defaults (bcc charges, GAFF2 atom types, all output formats)
acpype -i ligand.pdb

# From a MOL2 that already has good coordinates/atom names
acpype -i ligand.mol2

# From a SMILES string (needs OpenBabel); names the project "BUT"
acpype -i 'CCCC' -b BUT

# ABCG2 charges (this fork; needs AmberTools >= 24), explicit GAFF2
acpype -i ligand.pdb -c abcg2 -a gaff2
```

## Controlling charges

```bash
acpype -i ligand.pdb -c gas            # fast Gasteiger charges (no QM) — for testing
acpype -i ligand.pdb -c bcc            # AM1-BCC (default, production-quality)
acpype -i ligand.pdb -c abcg2 -a gaff2 # AM1-BCC-GAFF2 (improved hydration free energies)
acpype -i ligand.mol2 -c user          # keep the partial charges already in the MOL2
```

Give the net charge explicitly if guessing is unreliable, and the multiplicity for
radicals:

```bash
acpype -i anion.pdb -n -1
acpype -i radical.pdb -m 2
```

See [Charge Methods](Charge-Methods) for the trade-offs.

## Choosing the force field / atom type

```bash
acpype -i ligand.pdb -a gaff2     # GAFF2 (default)
acpype -i ligand.pdb -a gaff      # GAFF (original)
acpype -i ligand.pdb -a amber     # AMBER ff14SB atom types
acpype -i ligand.pdb -a amber2    # ff14SB + GAFF2
```

See [Atom Types & Force Fields](Atom-Types-and-Force-Fields).

## Choosing which topologies to write

```bash
acpype -i ligand.pdb -o gmx       # only GROMACS (+ AMBER, always produced)
acpype -i ligand.pdb -o cns       # only CNS/XPLOR
acpype -i ligand.pdb -o all       # everything (default)
```

## Converting an existing AMBER topology to GROMACS (amb2gmx)

```bash
acpype -p system.prmtop -x system.inpcrd
```

No charge/atom-type options apply here — it just reparses and re-emits. See
[amb2gmx Mode](amb2gmx-Mode).

## Useful operational flags

```bash
acpype -i ligand.pdb -f               # force recalculation (ignore cached output)
acpype -i big_ligand.pdb -s 7200      # raise the sqm/mopac time limit to 2 h
acpype -i ligand.pdb -d               # debug: keep temp files, verbose log
acpype -i ligand.pdb -w               # quiet: print nothing
acpype -i ligand.pdb -l               # sort atoms for GROMACS ordering
```

The full list is in the [Command-Line Reference](Command-Line-Reference).

## A typical end-to-end GROMACS workflow

```bash
acpype -i FFF.pdb                      # -> FFF.acpype/
cd FFF.acpype/
gmx grompp -c FFF_GMX.gro -p FFF_GMX.top -f em.mdp -o em.tpr
gmx mdrun -v -deffnm em
```

`em.mdp` and `md.mdp` run-parameter files are generated for you as starting points.
