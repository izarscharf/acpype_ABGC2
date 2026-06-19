# ACPYPE (ABCG2 fork) Wiki

**ACPYPE** — *AnteChamber PYthon Parser interfacE* — is a Python tool that uses
**Antechamber** (from AmberTools) to generate molecular-mechanics topologies and
parameters for small molecules / ligands, and converts them into the formats used by
**GROMACS**, **AMBER**, **CNS/XPLOR** and **CHARMM**. It can also convert an existing
AMBER topology (`prmtop`/`inpcrd`) straight to GROMACS — the classic *amb2gmx* job.

This wiki documents **this fork**, whose distinguishing feature is support for the
**`abcg2`** charge method (AM1-BCC-GAFF2, [He et al. 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC11948320/)),
implemented agentically with [Claude Code](https://claude.com/claude-code). See
[ABCG2 Charge Method](ABCG2-Charge-Method) for details.

> ℹ️ This is community/fork documentation. Upstream project:
> <https://github.com/alanwilter/acpype>. For the science, cite the papers listed in
> [`acpype/__init__.py`](../acpype/__init__.py) (ACPYPE: Sousa da Silva & Vranken,
> *BMC Res Notes* 2012).

## What ACPYPE does, in one diagram

```
                 small molecule (PDB / MOL2 / MDL / SMILES)
                                   │
                         ┌─────────▼──────────┐
                         │  ACTopol pipeline  │   (antechamber → tleap → parmchk2)
                         │  charges + GAFF/   │
                         │  AMBER parameters  │
                         └─────────┬──────────┘
                                   │  AMBER prmtop/inpcrd
                         ┌─────────▼──────────┐
   existing AMBER  ─────►│  MolTopol writers  │  (amb2gmx mode enters here)
   prmtop/inpcrd        │  format converters  │
                         └─────────┬──────────┘
            ┌───────────────┬──────┴───────┬─────────────────┐
          GROMACS         AMBER          CNS/XPLOR         CHARMM
       top/itp/gro/mdp  prmtop/inpcrd    top/par/inp      rtf/prm/inp
                        lib/frcmod                        (experimental)
```

## Start here

| If you want to… | Go to |
|---|---|
| Install on a laptop, server or **HPC/cluster** | [Installation](Installation) |
| Run your first conversion | [Quick Start](Quick-Start) |
| Look up a specific command-line flag | [Command-Line Reference](Command-Line-Reference) |
| Understand charge models (`gas`/`bcc`/`abcg2`/`user`) | [Charge Methods](Charge-Methods) |
| Choose an atom type / force field | [Atom Types & Force Fields](Atom-Types-and-Force-Fields) |
| Know which files you get and for which engine | [Input & Output Formats](Input-and-Output-Formats) |
| Convert an existing AMBER topology to GROMACS | [amb2gmx Mode](amb2gmx-Mode) |
| Drive ACPYPE from Python | [Python API](Python-API) |
| Understand the internals | [How ACPYPE Works](How-ACPYPE-Works) |
| Use the new GAFF2 charge model | [ABCG2 Charge Method](ABCG2-Charge-Method) |
| Hit a problem | [Limitations & Troubleshooting](Limitations-and-Troubleshooting) |

## At a glance

- **Inputs:** `.pdb`, `.mol2`, `.mdl`/`.mol`, or a **SMILES** string.
- **Charge methods:** `gas`, `bcc` *(default)*, **`abcg2`**, `user`.
- **Atom types / force fields:** `gaff`, `gaff2` *(default)*, `amber` (ff14SB), `amber2` (ff14SB + GAFF2).
- **Outputs:** GROMACS, AMBER, CNS/XPLOR, CHARMM (`-o all` by default).
- **Two modes:** ligand parameterisation (`-i`) and *amb2gmx* conversion (`-p`/`-x`).
- **Requirements:** Python 3.8+ (3.9+ recommended), AmberTools (Antechamber), and
  OpenBabel for PDB/SMILES input. `abcg2` additionally needs **AmberTools ≥ 24**.

## Key limitations (read before relying on output)

- **CHARMM output needs `charmmgen`**, which ships only with the bundled AmberTools
  binaries — it is **not** in conda-forge `ambertools`, so CHARMM topologies are
  unavailable in conda/git installs. See [Input & Output Formats](Input-and-Output-Formats).
- **`abcg2` requires AmberTools ≥ 24** and is validated against **GAFF2** atom types.
- **OPLS/AA GROMACS output is experimental** (a best-effort atom-type mapping).
- ACPYPE parameterises **one molecule/residue unit** at a time; it is not a
  whole-system / multi-component topology builder.
- *amb2gmx* with explicit **solvent** present is only partially supported
  (`-u/--direct` is experimental). See [amb2gmx Mode](amb2gmx-Mode).

A consolidated list lives in [Limitations & Troubleshooting](Limitations-and-Troubleshooting).
