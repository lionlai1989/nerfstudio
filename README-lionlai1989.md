# How to Install `nerfstudio` with CUDA Toolkit <strike>12.1</strike> 11.8

The official `nerfstudio` guide only supports CUDA 11.7 and 11.8 and uses `conda`, which I do not prefer. This tutorial
explains how to install `nerfstudio` using `venv` and CUDA <strike>12.1</strike> 11.8.

Note that after numerous experiments, I found `nerfstudio`, `tiny-cuda-nn` and `gsplat` only work together perfectly
with CUDA 11.8.

When working with the `PyTorch` package, it is generally unnecessary to install the system-wide CUDA Toolkit (e.g.,
`/usr/local/cuda-12.1/bin` and `/usr/local/cuda-12.1/lib64`) because PyTorch includes its own CUDA Toolkit libraries?

<strike>`nerfstudio` relies on [`tiny-cuda-nn`](https://github.com/NVlabs/tiny-cuda-nn), which can use the CUDA libraries
bundled with PyTorch. Therefore, ensure that your `PATH` and `LD_LIBRARY_PATH` do **not** include system-wide CUDA
Toolkit paths.</strike>

Note that `tiny-cuda-nn` relies on system-wide installed CUDA Toolkit 11.8. 

The first problem encountered is that I already have CUDA Toolkit 12.1 installed on my system path
`/usr/local/cuda-12.1`. The solution is to install CUDA Toolkit 11.8 on a different system path `/usr/local/cuda-11.8`
and make sure that the symlink `/usr/local/cuda` pointing to it.

```bash
# Export the RIGHT system-wide CUDA Toolkit path
export PATH=/usr/local/cuda-11.8/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-11.8/lib64:$LD_LIBRARY_PATH
```

## Steps to Install `nerfstudio` and `gsplat` with CUDA 11.8

1. Create a Python Virtual Environment

```
python3.10 -m venv venv_nerfstudio && source venv_nerfstudio/bin/activate
```

2. Update pip

It is crucial to constrain the pip version due to compatibility issues with the `tiny-cuda-nn` installation process:

>  DEPRECATION: tinycudann is being installed using the legacy 'setup.py install' method, because it does not have a
>  'pyproject.toml' and the 'wheel' package is not installed. pip 23.1 will enforce this behaviour change. A possible
>  replacement is to enable the '--use-pep517' option. Discussion can be found at
>  https://github.com/pypa/pip/issues/8559 

```
python3 -m pip install --upgrade "pip<23.1.0" "setuptools<70.0.0"
``` 

3. Install PyTorch

Install the PyTorch version compatible with CUDA 11.8:

```
python3 -m pip install "numpy<2.0.0" torch==2.1.2+cu118 torchvision==0.16.2+cu118 --extra-index-url https://download.pytorch.org/whl/cu118
```

4. Install `tiny-cuda-nn`

```
python3 -m pip install "numpy<2.0.0" ninja 'git+https://github.com/NVlabs/tiny-cuda-nn/#subdirectory=bindings/torch'
```

5. Install `nerfstudio`

Install `nerfstudio` in editable mode and ensure `numpy<2.0.0` is used to avoid compatibility issues:

```
python3 -m pip install -e . "numpy<2.0.0"
```

Note: Using `numpy>2.0.0` can lead to failures with compiled modules, as shown below:

```
A module that was compiled using NumPy 1.x cannot be run in
NumPy 2.2.0 as it may crash. To support both 1.x and 2.x
versions of NumPy, modules must be compiled with NumPy 2.0.
Some module may need to rebuild instead e.g. with 'pybind11>=2.12'.

If you are a user of the module, the easiest solution will be to
downgrade to 'numpy<2' or try to upgrade the affected module.
We expect that some modules will need time to support NumPy 2.
```

6. Test the Installation

Verify the installation by downloading test data and training a model:

```
# Download data
ns-download-data nerfstudio --capture-name=poster

# Train a model (requires ~30 GB of CPU memory for preparation)
ns-train nerfacto --data data/nerfstudio/poster
ns-train splatfacto --data data/nerfstudio/poster

# View 3DGS
ns-viewer --load-config outputs/poster/splatfacto/<datetime>/config.yml
```

## Keeping a Linear Git History with the Official `nerfstudio` Repository

To keep the fork aligned with the official `nerfstudio` repository, rebase the branch as follows:

1. Add the official repository as a remote:

    ```
    git remote add upstream https://github.com/nerfstudio-project/nerfstudio.git
    ```
2. Fetch updates from the official repository:

    ```
    git fetch upstream 
    ```

3. Rebase the latest upstream `main` branch onto the local `main` branch:

    ```
    git checkout main && git rebase upstream/main
    ```

4. Push the rebased branch to the fork:

    ```
    git push origin --force
    ```

References:
- [stackoverflow: How do I update or sync a forked repository on GitHub?
  ](https://stackoverflow.com/questions/7244321/how-do-i-update-or-sync-a-forked-repository-on-github)
- [stackoverflow: Git fork always commits ahead I don't
  want](https://stackoverflow.com/questions/72477056/git-fork-always-commits-ahead-i-dont-want)
