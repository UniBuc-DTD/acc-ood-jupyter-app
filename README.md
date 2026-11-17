# Open OnDemand Batch Connect - Jupyter Notebook Server

A Batch Connect app for the ACC's [Open OnDemand](https://www.openondemand.org/) instance that launches a Jupyter Notebook server within a Slurm batch job on the ACC HPC cluster.

The notebook server always runs from the shared **`tools/jupyter`** module — a read-only virtual environment on shared storage, loaded automatically inside the job.

## Form fields

| Field | Default | Meaning |
|---|---|---|
| Account | (from QOS) | Slurm accounting id / QOS |
| Partition | — | Slurm partition (e.g. `cpu-wide`, or `gpu` for GPUs) |
| Cores | `1` | CPU cores for the notebook (`sbatch -n`), 1–256 per node |
| Memory | `16G` | Memory for the job (`sbatch --mem`), Slurm units (`4G`, `32G`, `1T`, …) |
| GPUs | `0` | GPUs to allocate (`sbatch --gres=gpu:N`), 0–2; select partition `gpu` (hpc-nodegpu01, up to 2× NVIDIA H100 80 GB) |
| Hours | `1` | Wall time (the `gpu` partition allows up to 8 h) |
| Nodes | `1` | Number of nodes (the app is designed for single-node notebooks) |
| Virtualenv path | *(empty)* | Optional, see below |

## Using your own virtualenv as a notebook kernel

Leave **Virtualenv path** empty to use only the shared Jupyter venv, or give the absolute path to a virtualenv on shared storage, e.g. `/mnt/home/yourname/.venvs/myproj`.

For the virtualenv to work as a notebook kernel it must have **`ipykernel`** and **`venv-kernel`** installed (note the `setuptools<81` pin — `venv-kernel` needs the legacy `pkg_resources`, which newer setuptools removed):

```bash
/path/to/your/venv/bin/python -m pip install 'setuptools<81' ipykernel venv-kernel
```

When the job starts, the app registers that virtualenv as an additional Jupyter kernel (via `venv-kernel install`). You can then switch between the shared kernel and your virtualenv's kernel **per notebook** from the *Kernel → Change kernel* menu. The notebook server itself always keeps running from the shared `tools/jupyter` venv; your virtualenv only provides the kernel (and therefore its packages).

Notes:

- The virtualenv path must be reachable from the compute nodes (shared storage, e.g. `/mnt/home/...`).
- If the job log shows a `WARNING: 'venv-kernel' not found ...` message, the virtualenv is missing `venv-kernel` (install both packages as above).
- Kernel names default to the virtualenv's directory name.

## GPU workloads

Select partition `gpu` and set **GPUs** > 0. The Jupyter environment is CPU-only by default — for GPU computation install CUDA-enabled packages (e.g. `cupy`, `torch`, `tensorflow`) into a virtualenv on shared storage and select it via **Virtual environment path** (a GPU node's kernel is the only place `torch.cuda.is_available()` will be true).

## Session behaviour

- The job starts with the node-side login environment (`--export NONE`); the `module` function and your normal shell environment are available.
- The server listens on a random port with a random password; use the **Connect to Jupyter** button on the app's session page.
- Default wall time is 1 hour; the session page can be closed anytime (kills the job).
