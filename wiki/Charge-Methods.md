# Charge Methods (`-c`)

ACPYPE assigns partial atomic charges by handing the molecule to **Antechamber** with
a chosen charge method. The method is selected with `-c` and is reflected in the output
MOL2 filename: `<base>_<chargeType>_<atomType>.mol2`.

| `-c` value | Name | QM step? | Speed | Use for |
|---|---|---|---|---|
| `gas` | Gasteiger | no | instant | quick tests, sanity checks, geometry pipelines |
| `bcc` *(default)* | AM1-BCC | yes (AM1) | seconds–minutes | general production MD |
| `abcg2` | AM1-BCC-GAFF2 | yes (AM1) | seconds–minutes | production MD with **GAFF2**, better hydration free energies |
| `user` | read from MOL2 | no | instant | charges you computed elsewhere (RESP, QM, etc.) |

The AM1 pre-charge step (for `bcc`/`abcg2`) is run by the QM engine selected with `-q`
(`sqm` by default) and is bounded by the `-s` time limit.

---

## `gas` — Gasteiger

Empirical, topology-based partial charges. No quantum mechanics, so it is essentially
instantaneous and never fails on QM convergence.

- **Pros:** fast; deterministic; great for debugging a pipeline or when charges don't
  matter (e.g. you only need bonded parameters or geometry).
- **Limitations:** not suitable for quantitative MD — Gasteiger charges are crude
  compared with AM1-BCC. ACPYPE also uses a `gas` pass internally to **guess the net
  charge** when you don't pass `-n`.

## `bcc` — AM1-BCC (default)

Runs an AM1 semi-empirical calculation, then applies **bond charge corrections (BCC)**
to approximate HF/6-31G* RESP charges at a fraction of the cost. This is the standard,
well-validated choice for GAFF/GAFF2 small-molecule MD.

- **Pros:** good accuracy/cost balance; the de-facto default for ligand force fields.
- **Limitations:** requires a successful AM1 calculation (can be slow or fail to
  converge for large/strained molecules — raise `-s`, or adjust QM via `-k`). Charges
  depend on the input 3D geometry.

## `abcg2` — AM1-BCC-GAFF2 *(this fork)*

A **re-tuned BCC parameter set** layered on the same AM1-BCC machinery, fitted
specifically against **GAFF2** to improve hydration free energies
([He et al., *JCTC* 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC11948320/)). Same
workflow and cost as `bcc`; only the correction table differs.

- **Pros:** improved condensed-phase/solvation behaviour with GAFF2; drop-in
  replacement for `bcc`.
- **Limitations:**
  - **Requires AmberTools ≥ 24.** Older Antechamber (including the bundled
    AmberTools 21.11) reports *"Unknown charge method"*; ACPYPE turns this into a clear
    error telling you to upgrade.
  - **Validated against GAFF2 atom types.** Use with `-a gaff2` (default) or `-a amber2`;
    pairing it with `-a gaff`/`-a amber` triggers a warning (the result is not a
    validated combination).
- See the dedicated [ABCG2 Charge Method](ABCG2-Charge-Method) page for background,
  verification, and reference values.

## `user` — keep existing charges

Reads the partial charges already present in the input **MOL2** file and passes them
through unchanged.

- **Pros:** lets you supply RESP or high-level QM charges computed elsewhere.
- **Limitations:**
  - **MOL2 input only.** With a PDB there are no charges to read, so ACPYPE warns and
    **falls back to `bcc`** automatically.
  - You are responsible for charge quality and for the net charge being sensible.

---

## Picking a method

- Prototyping / does-it-run? → `gas`
- Standard GAFF2 MD → `bcc` (default) or **`abcg2`** if you have AmberTools ≥ 24 and
  care about solvation accuracy
- You already have trusted charges → `user` (MOL2)

## Related options

- `-n/--net_charge` — set the net charge explicitly (recommended for ions/charged
  species; otherwise guessed via Gasteiger).
- `-m/--multiplicity` — spin multiplicity for the QM step (radicals).
- `-q/--qprog`, `-k/--keyword` — QM engine and extra keywords for the AM1 step.
- `-s/--max_time` — abort the QM step if it exceeds this wall-time.
