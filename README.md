# Ascender + Singularity + sHCP for Jun

![stable](https://img.shields.io/badge/stable-v0.1.3-blue)
![python versions](https://img.shields.io/badge/python-3.8%20%7C%203.9-blue)
[![tests](https://github.com/cvpaperchallenge/Ascender/actions/workflows/lint-and-test.yaml/badge.svg)](https://github.com/cvpaperchallenge/Ascender/actions/workflows/lint-and-test.yaml)
[![MIT License](https://img.shields.io/github/license/cvpaperchallenge/Ascender?color=green)](LICENSE)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![Code style: flake8](https://img.shields.io/badge/code%20style-flake8-black)](https://github.com/PyCQA/flake8)
[![Imports: isort](https://img.shields.io/badge/%20imports-isort-%231674b1?style=flat&labelColor=ef8336)](https://pycqa.github.io/isort/)
[![Typing: mypy](https://img.shields.io/badge/typing-mypy-blue)](https://github.com/python/mypy)
[![DOI](https://zenodo.org/badge/466620310.svg)](https://zenodo.org/badge/latestdoi/466620310)

## What is Ascender?

Ascender (Accelerator of SCiENtific DEvelopment and Research) is a [GitHub repository template](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-template-repository) for research projects using Python as a developing language. The following features are pre-implemented to accelerate your development:

- **Container**: Use of [Docker](https://www.docker.com/) reduces development environment dependencies and improves code portability.
- **Virtual environment / package management**: Package management using [Poetry](https://python-poetry.org/) improves reproducibility of the same environment.
- **Coding style**: Automatic code style formatting using [Black](https://github.com/psf/black), [Flake8](https://github.com/pycqa/flake8), and [isort](https://github.com/PyCQA/isort).
- **Static type check**: Static type checking with [Mypy](https://github.com/python/mypy) to assist in finding bugs.
- **pytest**: Easily add test code using [pytest](https://github.com/pytest-dev/pytest).
- **GitHub features**: Some useful features, [workflow](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions) for style check and test for pull request, [issue template](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository), etc. are pre-implemented.

Please check [the slide format resources about Ascender (Japanese)](https://cvpaperchallenge.github.io/Britannica/ascender/ja) too.

## Project Organization

```
    ├── .github/           <- Settings for GitHub.
    │
    ├── data/              <- Datasets.
    │
    ├── models/            <- Pretrained and serialized models.
    │
    ├── container/         <- Singularity container.
    │
    ├── notebooks/         <- Jupyter notebooks.
    │
    ├── outputs/           <- Outputs.
    │
    ├── src/               <- Source code. This sould be Python module.
    │
    ├── tests/             <- Test codes.
    │
    ├── .flake8            <- Setting file for Flake8.
    ├── .dockerignore
    ├── .gitignore
    ├── LICENSE
    ├── container.sif      <- Singularity container file.
    ├── Makefile           <- Makefile used as task runner.
    ├── poetry.lock        <- Lock file. DON'T edit this file manually.
    ├── poetry.toml        <- Setting file for Poetry.
    ├── pyproject.toml     <- Setting file for Project. (Poetry, Black, isort, Mypy)
    └── README.md          <- The top-level README for developers.

```

## Prerequisites

- [Singularity](https://sylabs.io/)
- [Docker](https://www.docker.com/)

### Create Singularity Container

```bash
# Create sandbox from docker image
$ singularity build --sandbox container/ docker://nvcr.io/nvidia/pytorch:23.05-py
# Connect to the container's shell
singularity shell --writable container/
# Install poetry and whatever necessary
$ pip install poetry
# Convert the container into a SIF file
$ singularity build container.sif container/
# Install dependencies using poetry
$ singularity exec container.sif poetry install
# Load CUDA in case you are using sHPC
$ ml spider CUDA11.7.0

```

### Start development

```bash

# Run singularity container
$ singularity exec container.sif bash
# Create virtual environment and install dependent packages by Poetry
$ poetry install
```
Now, you are ready to start development with Ascender.

### Submit job file to sHPC
```bash
$ bsub < submission.sh
```

## FAQ

### Use Ascender without Docker

We recommend using Ascender with Docker as described above. However, you might not be able to install Docker in your development environment due to permission issues or etc.

In such cases, Ascender can be used without Docker. To do that, please install Poetry in your computer, and follow the steps describing in "Start development" section with ignoring the steps related to Docker.

```bash
# Install Poetry
$ pip3 install poetry

# Clone repo
$ git clone git@github.com:<YOUR_USER_NAME>/<YOUR_REPO_NAME>.git
$ cd <YOUR_REPO_NAME>

# Create virtual environment and install dependent packages by Poetry
$ poetry install
```

NOTE: CI job (GitHub Actions workflow) of Ascender is using Dockerfile. Therefore, using Ascender without Docker might raise error at CI job. In that case, please modify the Dockerfile appropriately or delete the CI job (`.github/workflows/lint-and-test.yaml`).

### Permission error is raised when execute `poetry install`.

Sometime `poetry install` might rise permission error like following:

```bash
$ poetry install
...

virtualenv: error: argument dest: the destination . is not write-able at /home/challenger/ascender
```

In that case, please check UID (user id) and GID (group id) at your local PC by following:

```bash
$ id -u $USER  # check UID
$ id -g $USER  # check GID
```

In Ascender, default value of both is `1000`. If UID or GID of your local PC is not `1000`, you need to modify the value of `UID` or `GID` inside of `docker-compose.yaml` to align your local PC (please edit their values from `1000`). Or if environmental variables `HOST_UID` and `HOST_GID` is defined at host PC, Ascender uses these values.

### Compatibility issue between PyTorch and Poetry

NOTE: Now poetry 1.2 is used in Ascender. So this issue is expected to be solved.

Currently, there is a compatibility issue between PyTorch and Poetry. This issue is being worked on by the Poetry community and is expected to be resolved in 1.2.0. (You can check pre-release of 1.2.0 from [here](https://github.com/python-poetry/poetry/releases/tag/1.2.0b3).)

We plan to incorporate Poetry 1.2.0 into Ascender immediately after its release. In the meantime, please consider using the workaround described in [this issue](https://github.com/python-poetry/poetry/issues/4231).

**Some related GitHub issues**

- https://github.com/python-poetry/poetry/issues/2339
- https://github.com/python-poetry/poetry/issues/2543
- https://github.com/python-poetry/poetry/issues/2613
- https://github.com/python-poetry/poetry/issues/3855
- https://github.com/python-poetry/poetry/issues/4231
- https://github.com/python-poetry/poetry/issues/4704

### Change the Python version to run CI jobs

By default, CI job (GitHub Actions workflow) of Ascender is run against Python 3.8 and 3.9. If you want to change the target Python version, please modify [the matrix part of `.github/workflows/lint-and-test.yaml`](https://github.com/cvpaperchallenge/Ascender/blob/master/.github/workflows/lint-and-test.yaml#L18).

### When changes to the Dockerfile are not reflected correctly on the image build

When you run `sudo docker compose up` after adding some modifications to the Dockerfile, you may find no changes have been made to the image built. In that case, please try following commands:

```shell
$ sudo docker compose build --no-cache
$ sudo docker compose up --force-recreate -d
```

When changes to the Dockerfile are not reflected, potential reasons are:

1. docker uses cache to build an image
1. docker doesn't recreate a container

`sudo docker compose build --no-cache` command build docker image with no cache (the solution for the 1st case). And `sudo docker compose up --force-recreate -d` command recreate and start containers (the solution for the 2nd case).

### Activate/deavtivate caching in CI job

Caching has been introduced in CI job (`lint-and-tests.yaml`) since `v0.1.2` to minimize latency due to Docker image build and Poetry install in the CI job.
However, this feature has not yet been fully tested, so if you do not want to use it in the CI job, please change the value of `USE_CACHE` variable in `lint-and-tests.yaml` to `false`.
