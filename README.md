[![Latest Github release](https://img.shields.io/github/release/nextgenusfs/funannotate2.svg)](https://github.com/nextgenusfs/funannotate2/releases/latest)
![Conda](https://img.shields.io/conda/dn/bioconda/funannotate2)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![Tests](https://github.com/nextgenusfs/funannotate2/actions/workflows/tests.yml/badge.svg)](https://github.com/nextgenusfs/funannotate2/actions/workflows/tests.yml)

# funannotate2: eukaryotic genome annotation pipeline

Funannotate2 is a comprehensive eukaryotic genome annotation pipeline that provides a complete workflow for annotating
eukaryotic genomes. It integrates various tools and databases to produce high-quality gene predictions and functional annotations.


### Quick start: Installation

#### Docker (recommended)

A pre-built image with all bioconda tooling (augustus, minimap2, miniprot, snap, glimmerhmm, diamond, trnascan-se, table2asn, …), `funannotate2`, `funannotate2-addons`, `helixerlite`, and pre-downloaded databases is published to Docker Hub and GHCR on each tagged release.

```shell
# pull the latest image (~8 GB; databases are baked in)
docker pull nextgenusfs/funannotate2:latest
# or from GHCR
docker pull ghcr.io/nextgenusfs/funannotate2:latest

# quick sanity check
docker run --rm nextgenusfs/funannotate2:latest funannotate2 --version
docker run --rm nextgenusfs/funannotate2:latest funannotate2 install -s

# run against a local data directory, persisting the BUSCO cache across runs
mkdir -p $PWD/data $PWD/busco_cache
docker run --rm -it \
    -v $PWD/data:/data \
    -v $PWD/busco_cache:/opt/busco_cache \
    -e BUSCO_DOWNLOAD_PATH=/opt/busco_cache \
    nextgenusfs/funannotate2:latest \
    funannotate2 predict -i /data/genome.fa -o /data/out --species "My species"
```

Notes:
- The image is `linux/amd64` only; on Apple Silicon it runs under Rosetta 2 emulation (Docker Desktop handles this automatically).
- BUSCO lineages are **not** bundled (~90 GB uncompressed) — mount a host directory as shown above so they download once and are reused.
- GeneMark is not included (license-restricted); install it locally and mount into the container if you need it.

#### Using pixi

If you prefer a native install without Docker, the repo ships a [pixi](https://pixi.sh/) workspace (`pixi.toml` / `pixi.lock`) that resolves the full environment on `linux-64`:

```shell
# install pixi once (see https://pixi.sh/latest/#installation)
curl -fsSL https://pixi.sh/install.sh | bash

# clone and install the locked environment
git clone https://github.com/nextgenusfs/funannotate2.git
cd funannotate2
pixi install --locked

# activate and install databases
pixi shell
export FUNANNOTATE2_DB=/path/to/funannotate2-db
funannotate2 install -d all
```

The pixi environment is currently defined for `linux-64` only. macOS users should use Docker.

#### Linux systems (conda)

Until this gets pushed to bioconda, can try this:
```shell
mamba create -n funannotate2 gfftk gapmm2 minimap2 miniprot snap "augustus==3.5.0" glimmerhmm diamond trnascan-se table2asn gb-io buscolite
conda activate funannotate2
python -m pip install git+https://github.com/nextgenusfs/funannotate2.git
```

#### Apple Silicon (M series)
Installation on apple silicon (M series) is a little bit more involved due to some dependency issues and non-native builds of some software.  I've not been able to find or build a version of `augustus` that will run, so instead I've been running `augustus` and `genemark` locally with Docker.  I've setup two repos with instructions on how to get this working (Need Docker Desktop installed) and then will need to put the bash wrapper files in your PATH to mimic the CLI interface.

https://github.com/nextgenusfs/dockerized-augustus

https://github.com/nextgenusfs/dockerized-genemark

Once that is working, you can then install most of the remaining dependencies with conda, although we need to leave out both `buscolite` and `funannotate2` because they have `augustus` as a dependency, instead we will install those python packages with pip.  The conda mkl<2022 is to avoid an annoying warning on apple silicon with the intel mkl package.

```shell
# first install most of the dependencies
mamba create -n funannotate2 --platform osx-64 "python>=3.7,<3.13" gfftk gapmm2 minimap2 miniprot snap glimmerhmm diamond trnascan-se gb-io pyhmmer pyfastx requests json-repair pytantan "mkl<2022"

# we can then add the required FUNANNOTATE2_DB env variable to the conda environment, note need to reactivate to use it
conda activate funannotate2
conda env config vars set FUNANNOTATE2_DB=/path/to/funannotate2-db
conda env config vars set AUGUSTUS_CONFIG_PATH=/path/to/augustus-3.5.0/config
conda deactivate

# now reactivate environment, and install the remaining python dependencies with pip
conda activate funannotate2
python -m pip install buscolite git+https://github.com/nextgenusfs/funannotate2.git

# now we can install the databases
funannotate2 install -d all
```

#### Other/Manual Installation

Additional tools like genemarkHMM must be installed manually due to licensing.

`funannotate2` is a python package, to install release versions use the pip package manager, like so:

```shell
pip install funannotate2
```
Or to install the bleeding edge version from github repo:

```shell
python -m pip install git+https://github.com/nextgenusfs/funannotate2.git
```

## Development

### Testing

Funannotate2 includes both unit tests and integration tests to ensure the code works correctly.

#### Running Tests

To run the tests, you need to install pytest and the package in development mode:

```bash
# Install pytest and coverage tools
pip install pytest pytest-cov

# Install funannotate2 in development mode
pip install -e .

# Run all tests
pytest

# Run with coverage report
pytest --cov=funannotate2

# Generate HTML coverage report
python scripts/run_coverage.py
```

For more information about testing, see the [TESTING.md](TESTING.md) file.

### Development Dependencies

To work on funannotate2 development, you'll need to install the development dependencies:

```shell
pip install pytest pytest-cov
```

### Documentation

Funannotate2 includes comprehensive documentation that covers installation, usage, API reference, and more. To build the documentation:

```bash
# Install Sphinx and the theme
pip install sphinx sphinx_rtd_theme

# Build the documentation
cd docs
make html
```

The built documentation will be in the `docs/_build/html` directory.

For more information about the documentation, see the [docs/README.md](docs/README.md) file.

### Running Tests

After installing the development dependencies, you can run the tests with:

```shell
python -m pytest
```

To run tests with coverage reporting:

```shell
python -m pytest --cov=funannotate2 --cov-report=term-missing
```

Or use the provided script to generate an HTML coverage report:

```shell
python scripts/run_coverage.py
```

To install the most up to date code from this repo, you can run:
```
python -m pip install git+https://github.com/nextgenusfs/funannotate2.git --upgrade --force --no-deps
```

### Citation

Funannotate2 includes a CITATION.cff file that provides citation information for the software. The version and release date in this file are automatically updated when a new release is created.

To cite funannotate2 in your work, you can use the citation information from the CITATION.cff file or generate a citation in your preferred format using tools like [citeas.org](https://citeas.org/).

---

## R-Cardenas Fork

This is a fork of [nextgenusfs/funannotate2](https://github.com/nextgenusfs/funannotate2), maintained at [R-Cardenas/funannotate2](https://github.com/R-Cardenas/funannotate2).

### Fixes applied

| File | Fix |
|------|-----|
| `utilities.py` | `ensure_busco_lineage()`: HEAD-probes the primary URL before downloading; falls back to odb10 if odb12 returns 4xx; renames the extracted directory to the expected path; emits a clear actionable error if all candidates fail |
| `utilities.py` | `normalize_busco_lineage_dir()`: converts a 3-column `lengths_cutoff` file (odb12 transitional format) to the 4-column odb10 format that buscolite expects |
| `predict.py` | Prefer `_odb10` BUSCO lineage over `_odb12` for all buscolite completeness calls — see [odb10 vs odb12](#odb10-vs-odb12) below |
| `predict.py` | Guard `mito_contigs.keys()` against `None` when the mitochondrial refseq database is not installed |
| `predict.py` | Guard all `/ float(stats["total"])` BUSCO coverage divisions against zero |
| `annotate.py` | Prefer `_odb10` BUSCO lineage for buscolite calls (same reason as predict.py) |
| `annotate.py` | Guard `pfam_search`, `dbcan_search`, `swissprot_blast`, and `merops_blast` return values against `None` when the annotation database is absent |
| `abinitio.py` | Guard `sum()/len()` divisions in `test_training()` against empty prediction lists |
| `train.py` | Guard `countGenesCDS / countGenes` against empty training gene dictionary |
| `train.py` | Initialise `values1/2/3` before loop in `getTrainResults()` to prevent `UnboundLocalError` on malformed output |
| `fastx.py` | Guard `maskLen / fa.size` and `gapLen / fa.size` against zero-size FASTA |
| `memory.py` | Guard `avg_rss_mb` and `avg_vms_mb` against empty sample lists |

### odb10 vs odb12

BUSCO lineage databases come in two generations:

- **odb10** — based on OrthoDB v10 (~2021). Smaller lineage sets (e.g. 758 marker genes for fungi) but well-supported by the version of `buscolite` bundled with funannotate2.
- **odb12** — based on OrthoDB v12 (~2023). Larger lineage sets (~1,019 marker genes for fungi, ~35% more) sourced from a much wider range of sampled genomes, giving more sensitive completeness estimates.

**Why this fork uses odb10 for completeness scoring:**

`buscolite` (the internal BUSCO library funannotate2 uses) was built for odb10 format. Real odb12 datasets cause two failure modes:

1. The `lengths_cutoff` file changed from 4 columns to 3 columns between odb10 and odb12 — buscolite's parser hard-codes 4-column unpacking and crashes.
2. odb12 HMM model names are not always present in the accompanying `scores_cutoff`, causing a `KeyError` when buscolite tries to look up a score for a hit it found.

The practical consequence is that odb12 datasets **cannot be used reliably with the current version of buscolite**. This fork detects whether an `_odb10` lineage directory is available alongside the `_odb12` one, and routes all buscolite calls to it. The `normalize_busco_lineage_dir()` function handles any remaining `lengths_cutoff` format mismatches as a safety net.

Completeness scores from odb10 are slightly less sensitive than odb12 would give, but they are accurate and stable. The gene models produced by the pipeline are unaffected — only the QC metric differs.

### Container usage

The fork is installed into the base `nextgenusfs/funannotate2` container via:

```dockerfile
FROM nextgenusfs/funannotate2:sha-1a7e335

RUN apt-get update && apt-get install -y --no-install-recommends git time && rm -rf /var/lib/apt/lists/*

RUN /app/.pixi/envs/default/bin/pip install buscolite==26.4.22

RUN /app/.pixi/envs/default/bin/pip install --no-deps \
    git+https://github.com/R-Cardenas/funannotate2@v26.6.9-mng-5

ENTRYPOINT []
```

BUSCO databases are pre-staged from S3 via Nextflow Fusion and pointed to using the `FUNANNOTATE2_DB` environment variable. When both `_odb10` and `_odb12` lineage directories are present under `FUNANNOTATE2_DB`, the pipeline uses odb12 for training and odb10 for buscolite completeness scoring automatically — no manual configuration needed.

