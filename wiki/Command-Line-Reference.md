# Command-Line Reference

Every option accepted by `acpype` (source of truth:
[`acpype/parser_args.py`](../acpype/parser_args.py)). Run `acpype -h` for the built-in
help. There are two mutually-exclusive operating modes:

- **Ligand mode** — give `-i` (input molecule). ACPYPE parameterises it.
- **amb2gmx mode** — give `-p` **and** `-x` (existing AMBER topology + coords) and
  **no** `-i`. ACPYPE only converts. See [amb2gmx Mode](amb2gmx-Mode).

Supplying both `-i` and `-p/-x`, or `-p` without `-x`, is an error.

---

## Input / output selection

| Flag | Long | Arg | Default | Description & limitations |
|---|---|---|---|---|
| `-i` | `--input` | file or SMILES | — | Input: `.pdb`, `.mol2`, `.mdl`/`.mol`, or a SMILES string. PDB and SMILES require **OpenBabel**. Mandatory unless `-p`/`-x` are used. |
| `-b` | `--basename` | name | input basename | Project name → output folder `<name>.acpype/` and file prefixes. |
| `-p` | `--prmtop` | file | — | AMBER `prmtop` for *amb2gmx* mode; always used with `-x`. |
| `-x` | `--inpcrd` | file | — | AMBER `inpcrd` for *amb2gmx* mode; always used with `-p`. |
| `-o` | `--outtop` | `all`/`gmx`/`cns`/`charmm` | `all` | Which topologies to write. **AMBER files are always produced** regardless. `charmm` needs `charmmgen` (see [Formats](Input-and-Output-Formats)). |

## Charges & chemistry

| Flag | Long | Arg | Default | Description & limitations |
|---|---|---|---|---|
| `-c` | `--charge_method` | `gas`/`bcc`/`abcg2`/`user` | `bcc` | Charge model. `abcg2` needs **AmberTools ≥ 24** and is validated for GAFF2. `user` reads charges from the input **MOL2** only. See [Charge Methods](Charge-Methods). |
| `-n` | `--net_charge` | int | guessed | Net molecular charge. If omitted, ACPYPE guesses it via Gasteiger charges; give it explicitly when guessing drifts. |
| `-m` | `--multiplicity` | int | `1` | Spin multiplicity (2S+1). Use `2` for a radical, etc. |
| `-a` | `--atom_type` | `gaff`/`gaff2`/`amber`/`amber2` | `gaff2` | Atom typing / force field. See [Atom Types](Atom-Types-and-Force-Fields). `abcg2` should be paired with `gaff2`/`amber2`. |
| `-q` | `--qprog` | `sqm`/`divcon`/`mopac` | `sqm` | QM engine for the AM1 step of `bcc`/`abcg2`. `sqm` ships with AmberTools; `mopac`/`divcon` must be installed separately. |
| `-k` | `--keyword` | "quoted" | — | Extra `sqm`/`mopac` keywords, e.g. `-k "qm_theory='AM1', grms_tol=0.0005, ..."`. Advanced; overrides the defaults for that QM run. |

## Output tuning

| Flag | Long | Default | Description & limitations |
|---|---|---|---|
| `-z` | `--gmx4` | off | Write Ryckaert–Bellemans dihedrals in the **old GROMACS 4.0** style. Only needed for legacy GROMACS. |
| `-t` | `--cnstop` | off | Write CNS topology with **allhdg-like** parameters (**experimental**). |
| `-g` | `--merge` | off | Merge lower/upper-case atom types in the GMX `top` when their parameters are identical (smaller files). |
| `-l` | `--sorted` | off | Sort atoms for GROMACS ordering. |
| `-j` | `--chiral` | off | Create improper-dihedral parameters for chiral atoms in **CNS** output. |
| `-u` | `--direct` | off | *amb2gmx* only: direct conversion for any solvent (**experimental**; see notes). Error if used outside amb2gmx mode. |

## Execution control

| Flag | Long | Default | Description & limitations |
|---|---|---|---|
| `-f` | `--force` | off | Recompute topologies from scratch, ignoring cached output in the `.acpype` folder. |
| `-s` | `--max_time` | `10800` (3 h) | Max wall-time (seconds) for the `sqm`/`mopac` step before ACPYPE aborts it (via `SIGALRM`). Raise for large molecules. |
| `-d` | `--debug` | off | Keep all temporary files and emit verbose logs. **Mutually exclusive with `-w`.** |
| `-w` | `--verboseless` | off | Print nothing. **Mutually exclusive with `-d`.** |
| `-y` | `--ipython` | off | Drop into an IPython shell after running (needs `ipython` installed) — for interactive inspection of the molecule object. |
| `-v` | `--version` | — | Print the version banner and exit. |

---

## Notes

- **Caching:** ACPYPE reuses an existing `<basename>.acpype/` if present. Use `-f`
  after changing inputs/options to avoid stale results.
- **Exit codes:** `0` on success, `19` on failure (the CLI catches exceptions and logs
  `ACPYPE FAILED: <reason>`).
- **Guessed net charge:** computed with a quick Gasteiger (`-c gas`) pass regardless of
  the final charge method; a large non-integer "charge drift" triggers a warning and,
  unless `-f` is set, an error — pass `-n` to be explicit.
- **`-q mopac`/`divcon`:** ACPYPE will build the command, but these external programs
  are not bundled; without them the QM step fails. `sqm` (default) is the supported path.
