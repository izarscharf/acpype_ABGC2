# Running this (modified) ACPYPE on a new machine / HPC

This fork adds the **`abcg2`** (AM1-BCC-GAFF2) charge method. `abcg2` needs
**`AmberTools >= 24`**, which the binaries bundled in the official `pip`/`docker`
builds do **not** provide — so on a fresh machine or cluster you get the tools from
`conda-forge` and run **this checkout's** code (not the PyPI `acpype`, which is the
unmodified upstream version).

Everything below works without `root`/`sudo`.

---

## 1. Get conda

Most HPC systems provide it as a module; otherwise install Miniforge into your home
or scratch space.

**If your site already provides it as a module**, just load it:

```bash
module load miniconda      # or: module load anaconda / miniforge (name varies)
```

**Otherwise, install Miniforge yourself** (no admin needed). `cd` to where you want
it to live first — on HPC prefer a project/scratch area over `$HOME` if quota is
tight:

```bash
cd /path/to/where/you/want/it          # e.g. $SCRATCH or a project dir

# download the latest Linux x86_64 installer:
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh

# verify the download — compare this against the published hash on the release page
# (https://github.com/conda-forge/miniforge/releases):
sha256sum Miniforge3-Linux-x86_64.sh

# install non-interactively into ./miniforge3 (relative to the current dir):
bash Miniforge3-Linux-x86_64.sh -b -p ./miniforge3
```

Activate it for the current shell:

```bash
source ./miniforge3/etc/profile.d/conda.sh
```

Optionally, make `conda` available in every new shell, but **without** auto-activating
the `base` env (recommended on HPC so it doesn't slow logins or shadow site tools):

```bash
./miniforge3/bin/conda init bash            # adds a block to ~/.bashrc
conda config --set auto_activate_base false # don't auto-enter base on login
```

> On shared filesystems, point conda's package/env storage at a larger quota, e.g.
> `export CONDA_PKGS_DIRS=$SCRATCH/conda/pkgs` and create envs with `-p
> $SCRATCH/envs/acpype` if your `$HOME` is small.

## 2. Clone and create the environment

```bash
git clone <this-repo-url> acpype
cd acpype

# Creates an env named 'acpype' with a modern AmberTools (>=24, abcg2-capable)
# and Open Babel from conda-forge:
conda env create -f environment.yml
conda activate acpype
```

## 3. Install THIS checkout (important)

`environment.yml` pip-installs `acpype` from PyPI, which is the **upstream** version
**without** `abcg2`. Overlay this modified checkout so the `acpype` command runs the
local code:

```bash
pip install -e .          # editable install of the current clone
```

(If you prefer not to install at all, you can instead run the local script directly
**from the clone's top directory**: `./run_acpype.py ...`. Outside that directory
you'd need `PYTHONPATH=/path/to/acpype`. The `pip install -e .` route is simpler
because the `acpype` command then works from anywhere in the env.)

## 4. Verify the toolchain

```bash
# the modified acpype is on PATH:
acpype -v

# AmberTools is recent and exposes abcg2:
antechamber -L | grep -i abcg2          # -> "ABCG2  abcg2  8"

# end-to-end smoke test (produces benzene_abcg2_gaff2.mol2):
acpype -i tests/benzene.pdb -c abcg2 -a gaff2
```

If `antechamber -L` does **not** list `abcg2`, your env resolved an older
`AmberTools`; recreate it with `conda env create -f environment.yml` (which pins
`ambertools >=24`).

## 5. Run

```bash
acpype -i my_ligand.pdb -c abcg2 -a gaff2     # ABCG2 / GAFF2
acpype -i my_ligand.mol2                       # defaults: bcc / gaff2
acpype -h                                      # all options
```

Output lands in `<basename>.acpype/` in the current directory.

`acpype` finds `antechamber`, `tleap`, `parmchk2` and `obabel` automatically as
long as they're on `PATH` (they are, once the env is active) — you do **not** need to
set `AMBERHOME` or use the bundled binaries.

## 6. Submitting as a batch job (SLURM example)

`acpype` is a short serial job; the only compute-heavy step is the AM1 charge
calculation (`sqm`), which is single-threaded. A modest allocation is plenty.

```bash
#!/bin/bash
#SBATCH --job-name=acpype
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --time=00:30:00
#SBATCH --mem=4G

# make conda available — match how you set it up in step 1:
module load miniconda                                   # if it's a site module, OR
# source /path/to/miniforge3/etc/profile.d/conda.sh     # if you installed it yourself
conda activate acpype                                    # or: conda activate /path/to/env

cd "$SLURM_SUBMIT_DIR"
acpype -i my_ligand.pdb -c abcg2 -a gaff2
```

Submit with `sbatch run_acpype.slurm`. For very large molecules, raise `--time`
(and `acpype`'s own `-s` max-time tolerance, default 3 h).

---

## Notes & limitations

- **CHARMM topologies** rely on the `charmmgen` binary, which the conda-forge
  `ambertools` package does **not** ship. Via this conda setup, CHARMM output is
  unavailable; GROMACS, AMBER and CNS/XPLOR topologies are produced normally.
- **`abcg2` is validated against GAFF2 atom types** — use it with `-a gaff2`
  (default) or `-a amber2`. `acpype` warns if you pair it with `-a gaff`/`-a amber`.
- Create the conda env **once** on a shared filesystem and reactivate it in each job;
  no need to rebuild per run.
- See [`docs/abcg2/`](docs/abcg2/) for the ABCG2 implementation details and the
  "Using ABCG2 Charges" section of the [README](README.md) for background.
