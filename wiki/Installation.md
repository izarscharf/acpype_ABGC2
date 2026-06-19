# Installation

ACPYPE itself is a **pure-Python** package. Its only runtime requirements are
**Antechamber** (from `AmberTools`) and, for PDB/SMILES input, **Open Babel**. The
cleanest way to provide those is `conda` from `conda-forge`, which gives you a current
`AmberTools` so **every** charge method — including [`abcg2`](ABCG2-Charge-Method) —
works without relying on any binaries bundled inside the package.

- Python **3.8+** is required (3.9+ recommended).
- `abcg2` additionally requires **AmberTools ≥ 24** (conda-forge's `ambertools`
  currently resolves to 24.8, which supports it).

> **Important for this fork:** the `acpype` package on **PyPI/conda-forge is the
> upstream version without `abcg2`**. To use the ABCG2 feature you must run **this
> checkout's code**. The provided [`environment.yml`](../environment.yml) installs the
> local checkout for you (`-e .`); if you set up the env another way, do
> `pip install -e .` from the repo root (or run `./run_acpype.py`). Symptom of running
> the wrong build: `acpype -c abcg2` fails with
> `invalid choice: 'abcg2' (choose from gas, bcc, user)`. See
> [Run this fork](#5-run-this-fork).

---

## 1. Conda (recommended)

Pulls `AmberTools` + `OpenBabel` from conda-forge — fully functional, modern toolchain:

```bash
conda install -c conda-forge acpype
```

## 2. From the provided `environment.yml` (reproducible, from a clone)

Creates a self-contained env named `acpype` with a modern, abcg2-capable `AmberTools`
(`>= 24`), `OpenBabel`, and **this checkout installed editable** (so the `acpype`
command runs the fork, not the PyPI build). Run it **from the repo root** — the env file
uses `-e .`:

```bash
git clone https://github.com/izarscharf/acpype_ABGC2.git acpype_ABGC2
cd acpype_ABGC2
conda env create -f environment.yml
conda activate acpype
```

See [`environment.yml`](../environment.yml). Nothing else is needed for `abcg2` to work.

## 3. PyPI (pure Python; bring your own AmberTools)

```bash
# get the 3rd-party tools first (a current, abcg2-capable AmberTools):
conda create -n acpype -c conda-forge ambertools openbabel
conda activate acpype
pip install acpype
```

The PyPI/Docker artifacts additionally **embed** a stripped `AmberTools v.21.11`
(Linux Ubuntu20 / macOS **Intel** only) that ACPYPE uses **only when no `antechamber`
is on your `PATH`**. That fallback does **not** support `abcg2` and may hit
library-dependency issues — so prefer the conda-provided tools above.

## 4. Docker

```bash
ln -fsv "$PWD/acpype_docker.sh" /usr/local/bin/acpype_docker   # Linux/macOS
acpype_docker -i CCCC
acpype_docker -i tests/DDD.pdb -c gas
```

The Docker image is batteries-included but, like the bundled binaries, currently
ships AmberTools 21.11 (no `abcg2`).

## 5. Run this fork

The provided `environment.yml` (method 2) already installs this checkout editable, so
`acpype` runs the fork — no extra step. But if you installed `acpype` from **PyPI**
(method 3) or built the env another way, that's the *upstream* build without `abcg2`;
overlay the local code from the repo root:

```bash
cd acpype_ABGC2           # the clone
pip install -e .          # editable install — `acpype` now runs this checkout
```

…or, without installing, run from the repo's top directory:

```bash
./run_acpype.py -i tests/benzene.pdb -c abcg2 -a gaff2
```

Verify everything is wired up:

```bash
acpype -v                          # version banner of this checkout
antechamber -L | grep -i abcg2     # -> "ABCG2  abcg2  8" (AmberTools >= 24)
acpype -i tests/benzene.pdb -c abcg2 -a gaff2   # end-to-end smoke test
```

---

## HPC / cluster installation

The repository ships a dedicated, copy-pastable guide for fresh machines and clusters
(no `root` needed): **[`HPC_SETUP.md`](../HPC_SETUP.md)**. It covers, in order:

1. **Getting conda** — via a site `module load`, or a self-contained Miniforge install:

   ```bash
   cd /path/to/where/you/want/it          # prefer $SCRATCH / project space
   wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh
   sha256sum Miniforge3-Linux-x86_64.sh   # compare to the hash on the release page
   bash Miniforge3-Linux-x86_64.sh -b -p ./miniforge3
   source ./miniforge3/etc/profile.d/conda.sh
   # optional, recommended on HPC:
   ./miniforge3/bin/conda init bash
   conda config --set auto_activate_base false
   ```

2. **Clone + `conda env create -f environment.yml`** (from the repo root) — installs a
   modern, abcg2-capable toolchain **and** this checkout editable, so `acpype` runs the
   fork. (If you build the env another way, run `pip install -e .` from the repo root, or
   you'll silently get upstream `acpype` with no `abcg2`.)
3. **Verifying** the toolchain (`antechamber -L | grep abcg2`).
4. **Running**, including where output lands and why `AMBERHOME` need not be set.
5. **A SLURM batch-job template** (ACPYPE is a short serial job; only the AM1/`sqm`
   charge step is compute-bound, and it is single-threaded).

Refer to [`HPC_SETUP.md`](../HPC_SETUP.md) for the full, current text — it is the
canonical source and this page intentionally mirrors rather than duplicates it.

> **Storage tip (HPC):** if `$HOME` quota is small, set
> `export CONDA_PKGS_DIRS=$SCRATCH/conda/pkgs` and create envs under scratch
> (`conda env create -f environment.yml -p $SCRATCH/envs/acpype`).
