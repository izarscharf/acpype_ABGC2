# How ACPYPE Works (Architecture)

This page describes the internal pipeline and the main functions, so you can read logs,
debug failures, and understand where each output comes from. Source:
[`acpype/topol.py`](../acpype/topol.py), [`acpype/cli.py`](../acpype/cli.py).

## Modules

| Module | Role |
|---|---|
| `cli.py` | Argument handling and orchestration (`init_main`); chooses ligand vs amb2gmx mode. |
| `parser_args.py` | Defines all command-line options. |
| `topol.py` | The engine: `AbstractTopol` (shared logic + writers), `ACTopol` (ligand mode), `MolTopol` (parse AMBER + amb2gmx), `Topology_14` (1-4 scaling). |
| `acs_api.py` | Files-in-memory JSON wrapper (`acpype_api`). |
| `params.py` | Constants, templates (`tleap`, mdp), atom-type maps, output file lists. |
| `utils.py` | Helpers: binary discovery (`find_bin`), `set_for_pip` (bundled-binary fallback), process/timeout helpers. |
| `mol.py` | Lightweight `Atom`/`AtomType` data classes. |
| `logger.py` | Logging configuration and log capture. |

## The two modes (from `init_main`)

- **Ligand mode** (`-i`): build `ACTopol` → `createACTopol()` → `createMolTopol()`.
- **amb2gmx mode** (`-p`+`-x`, no `-i`): build `MolTopol(amb2gmx=True)` →
  `writeGromacsTopolFiles()`.

On exit, temp dirs are cleaned (unless `-d`), the log is copied next to the output, and
the process returns `0` (ok) or `19` (failed).

## Ligand pipeline, step by step

```
ACTopol.__init__
  ├─ locate antechamber / obabel (set_for_pip falls back to bundled binaries
  │   only if antechamber is not already on PATH)
  ├─ if SMILES: convertSmilesToMol2()  (OpenBabel/pybel)
  ├─ setResNameCheckCoords()           (residue name + coordinate sanity checks)
  └─ guessCharge()                     (Gasteiger pass if -n not given)

createACTopol()
  ├─ execAntechamber()  → assigns charges (-c) and atom types (-a):
  │     antechamber -c <method> -at <type> -df <qprog> ...   → <base>_<c>_<at>.mol2
  ├─ execParmchk()      → parmchk2 → <base>_AC.frcmod (missing/estimated params)
  └─ execTleap()        → tleap → <base>_AC.prmtop / .inpcrd / .lib

createMolTopol()
  ├─ MolTopol parses the AMBER prmtop/inpcrd into objects:
  │     getAtoms, getBonds, getAngles, getDihedrals, getChirals, getCoords, getABCOEFs
  ├─ balanceCharges()                  (enforce integer net charge)
  └─ writers, per -o:
        writeGromacsTopolFiles()  (+ writeGroFile, writePosreFile, writeMdpFiles, OPLS)
        writeCnsTopolFiles()
        writeCharmmTopolFiles()   (needs charmmgen)
        writePdb()                (<base>_NEW.pdb)
```

### Key functions

- **`guessCharge()`** — runs a quick `-c gas` pass to estimate the net charge when `-n`
  is absent; warns/errors if the result drifts far from an integer (override with `-n`
  or `-f`).
- **`execAntechamber(chargeType, atomType)`** — builds and runs the generic
  `antechamber` command. The charge method and atom type flow straight through, which is
  why `abcg2` needed no special-casing here (only the allow-list in `parser_args.py`).
  Bounded by a `SIGALRM` timer set from `-s` (`signal_handler` kills a runaway `sqm`).
- **`execParmchk()` / `checkFrcmod()`** — generate and sanity-check the `frcmod`.
- **`execTleap()` / `checkLeapLog()`** — assemble the AMBER topology; parse the leap log
  for errors.
- **`balanceCharges()`** — adjusts charges so the total is an exact integer.
- **`setProperDihedralsCoef()` / `Topology_14`** — dihedral coefficients and 1-4
  scaling (including non-uniform 1-4 factors, e.g. GLYCAM06).
- **Writers** (`writeGromacsTop`, `writeCnsTopolFiles`, `writeCharmmTopolFiles`,
  `writeGroFile`, `writeMdpFiles`, `writePosreFile`, `writePdb`) — format-specific
  emission from the parsed model.

## Tool discovery & the bundled-binary fallback

`set_for_pip()` (in `utils.py`) only prepends the bundled
`acpype/amber_linux|amber_macos` binaries to `PATH`/`AMBERHOME` **when no `antechamber`
is found on your `PATH`**. So in any conda env that provides AmberTools, the bundled
binaries are never used — which is exactly how a modern, `abcg2`-capable AmberTools gets
picked up. See [Installation](Installation).

## Error handling & timeouts

- Each major step is wrapped; a failure logs `ACPYPE FAILED: <reason>` and exits `19`.
- The QM charge step is time-limited by `-s` (default 3 h); on timeout the `sqm`/`mopac`
  process tree is killed and the run aborts with a clear message.
- `-d/--debug` keeps the temporary folder and emits verbose logs for diagnosis.

For what can go wrong and how to fix it, see
[Limitations & Troubleshooting](Limitations-and-Troubleshooting).
