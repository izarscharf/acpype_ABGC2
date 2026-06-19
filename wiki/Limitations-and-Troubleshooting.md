# Limitations & Troubleshooting

## Known limitations (consolidated)

### Scope
- **One molecule / residue unit per run.** ACPYPE parameterises a single ligand or
  residue, not whole multi-component systems or full proteins. Assemble systems with
  `tleap`/`pdb2gmx` and use ACPYPE for the small-molecule part.

### Charges
- **`abcg2` requires AmberTools ≥ 24** and is validated only for **GAFF2** atom types
  (`-a gaff2`/`amber2`). See [ABCG2](ABCG2-Charge-Method).
- **`bcc`/`abcg2` depend on a successful AM1 (`sqm`) calculation** — large/strained
  molecules can be slow or fail to converge.
- **`-c user` needs MOL2 input** (no charges to read from a PDB → falls back to `bcc`).
- **Net charge is guessed** (Gasteiger) unless `-n` is given; charged species should set
  `-n` explicitly.

### Output formats
- **CHARMM output requires `charmmgen`**, present only in the bundled AmberTools
  binaries — **not** in conda-forge `ambertools`. CHARMM topologies are therefore
  unavailable in conda/git installs.
- **OPLS/AA GROMACS output is experimental** (heuristic atom-type mapping; verify before
  use).
- **CNS `-t` (allhdg) and `-j` (chiral impropers)** are experimental.

### amb2gmx
- **Explicit solvent is only partially supported**; `gmx mdrun` setups with solvent may
  be incomplete. `-u/--direct` is experimental. See [amb2gmx Mode](amb2gmx-Mode).

### Distribution
- **Bundled binaries are Intel-only** (Linux Ubuntu20 / macOS Intel) and may have
  library-dependency issues — prefer conda-provided AmberTools, especially on Apple
  Silicon or HPC.
- **This fork's `abcg2` code is not in the PyPI/conda `acpype` package** — run the
  checkout (`pip install -e .` or `./run_acpype.py`). See [Installation](Installation).

---

## Troubleshooting

### "Unknown charge method (abcg2)"
Your `antechamber` predates ABCG2. Install **AmberTools ≥ 24**
(`conda install -c conda-forge ambertools`) and confirm with
`antechamber -L | grep abcg2`. See [ABCG2](ABCG2-Charge-Method).

### `-c abcg2` rejected by the argument parser
You're running the **upstream** `acpype`, not this fork. Do `pip install -e .` from the
clone, or run `./run_acpype.py`. See [Installation](Installation#5-run-this-fork).

### "Missing ANTECHAMBER" / no executable
`antechamber` isn't on `PATH` and no bundled fallback applies. Activate the conda env
(`conda activate acpype`) or set up AmberTools. Check `AMBERHOME` and `which antechamber`
(note: shell `alias` does not help ACPYPE).

### "No module named 'openbabel'"
PDB/SMILES input needs the **OpenBabel Python module** in the *same* environment as
ACPYPE — the `obabel` binary alone is not enough. Install `openbabel`
(conda) or `openbabel-wheel` (pip) into that env. MOL2/MDL input avoids this.

### Charge-drift error / "Error with calculated charge"
The guessed net charge is far from an integer. Pass the correct **`-n`**, or use `-f`
to proceed anyway. Check your input's protonation/connectivity.

### `sqm` times out ("Semi-QM taking too long… aborting!")
The AM1 step exceeded `-s` (default 3 h). Raise it (`-s 21600`), simplify/clean the
input geometry, or tune QM via `-k`.

### Missing/penalised parameters
Inspect `<base>_AC.frcmod`. `parmchk2` fills gaps with estimates; large penalties or
`ATTN` markers mean you should review or refit those terms before production runs.

### CHARMM files not produced
Expected with conda/git installs (no `charmmgen`). Use the pip/Docker bundle if you need
CHARMM, or generate CHARMM parameters by another route.

### Inspecting a failed run
Re-run with **`-d/--debug`**: temporary files are kept and the log is verbose. The log
is also copied next to the output folder. For interactive poking, `-y` opens IPython
with the `molecule` object in scope.

---

## Getting more detail

- Internals and the function-by-function pipeline: [How ACPYPE Works](How-ACPYPE-Works).
- Per-flag behaviour: [Command-Line Reference](Command-Line-Reference).
- Upstream issues: <https://github.com/alanwilter/acpype/issues>.
