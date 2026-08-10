# Avianplumage_Birdimage
Open Bird Image Pipeline A Python command-line pipeline for discovering and downloading openly licensed bird photographs from iNaturalist.

Orders are read from a plain-text file, images are validated before storage, and each successful download receives taxonomy, observation, licensing, attribution, and checksum metadata.

The source-qualified identifier inaturalist-photo-<photo_id>prevents duplicate downloads across repeated runs. This project does not download Macaulay Library media or attach Macaulay metadata to unrelated images.

Features
Discovers bird taxa by scientific order.

Accepts only CC0 and CC BY photographs served from the iNaturalist Open Dataset host.

Rejects missing, noncommercial, no-derivatives, and all-rights-reserved licenses.

Validates JPEG files before moving them into the output directory.

Resumes safely without duplicating previously downloaded images.

Replaces missing or corrupt files through temporary .partfiles and atomic moves.

Records metadata in CSV and append-only event history in JSONL.

Supports dry runs, bounded downloads, and per-species limits.

Includes an optional workflow for reviewing reviewed images for Diffusion Autoencoder training.

Licensing and attribution
The pipeline preserves the licensing and attribution information supplied by each iNaturalist image. Keep metadata.csvwith downloaded images and preserve the following fields whenever images are used or redistributed:

Creator

Attribution

Source URL

License code

License URL

Photographers retain the rights described by the license attached to each image. The repository intentionally does not declare a source-code license; choose one before public publication if appropriate.

Do not use this tool to bypass provider controls, download restricted media, or discard attribution and licensing obligations.

Requirements
Python 3.11 or newer

Git

pipxfor the recommended isolated installation

No additional runtime binaries such as FFmpeg, ExifTool, ImageMagick, or a browser driver are required.

Install Python and Git if they are not already installed.

Installation
Recommended: pipx
Windows PowerShell
PowerShell
py -3 --version
py -m pip install --user pipx
py -m pipx ensurepath
Confirm that py -3 --versionreports Python 3.11 or newer. Restart PowerShell after ensurepath, then install and verify the application:

PowerShell
pipx install git+https://github.com/chpngyu/open-bird-image-pipeline.git
bird-image-download --help
macOS
Install Homebrew if needed, then run:

bash
brew install python git pipx
python3 --version
pipx ensurepath
Confirm that python3 --versionreports Python 3.11 or newer. Restart the terminal after ensurepath, then install the application:

bash
pipx install git+https://github.com/chpngyu/open-bird-image-pipeline.git
bird-image-download --help
Ubuntu/Debian
bash
sudo apt update
sudo apt install python3 python3-venv python3-pip git pipx
python3 --version
pipx ensurepath
The distribution-provided Python may be older than 3.11. If so, install a supported version using the official Python documentation or a newer distribution release, then point pipxand virtual-environment commands to that executable.

On systems with multiple Python versions, pass a verified Python 3.11-or-newer executable to pipxwith --python.

Developer installation
Clone the repository and change into its directory.

Windows PowerShell
PowerShell
py -3 --version
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -e ".[test]"
bird-image-download --help
python -m pytest -q
macOS, Linux, and other POSIX shells
bash
python3 --version
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -e ".[test]"
bird-image-download --help
python -m pytest -q
Order file
Create a UTF-8 text file with one bird order per line. Blank lines and comments beginning with #are ignored. Duplicate names are removed case-insensitively.

text
# Perching birds
Passeriformes

# Shorebirds
Charadriiformes
The included orders.txtstarts with Passeriformes.

Usage
Interactive run
When no count option is supplied in an interactive terminal, the command asks for the total number of images across all listed orders:

PowerShell
bird-image-download `
  --orders-file orders.txt `
  --output inaturalist_images
For POSIX shells:

bash
bird-image-download \
  --orders-file orders.txt \
  --output inaturalist_images
The default output directory is inaturalist_imagesrelative to the current directory, so --outputmay be omitted when running from the intended project root.

Automated runs
Preview three images without downloading:

bash
bird-image-download --orders-file orders.txt --output inaturalist_images --limit 3 --dry-run
Download three images in total:

bash
bird-image-download --orders-file orders.txt --output inaturalist_images --limit 3
Request a bounded number per discovered species:

bash
bird-image-download --orders-file orders.txt --output inaturalist_images --images-per-species 2 --dry-run
Noninteractive processes must provide either --limitor --images-per-species; otherwise the command exits instead of waiting for input.

Start live smoke tests with --dry-runand a small --limit.

Output
A typical output directory looks like this:

text
inaturalist_images/
├── images/
│   └── Passeriformes/
│       └── Corvus_corax_inaturalist-photo-12345.jpg
├── download_manifest.jsonl
├── metadata.csv
└── run_summary.json
metadata.csv
Contains the latest successful record for each source-qualified image UID, including:

Order and family, when available

Scientific and common names

iNaturalist taxon, observation, and photo identifiers

Description, observed date, public place text, quality grade, captive/cultivated state, annotations, and tags

Source URLs, creator, license, and attribution

Image dimensions and SHA-256 checksum

Local path, status, and retrieval time

download_manifest.jsonl
An append-only event history for download and resume decisions.

run_summary.json
Describes the latest execution, including the run ID, requested orders and count, dry-run state, timestamps, discovery and selection totals, downloaded/skipped/failed counts, and final status.

Resume behavior
Before downloading, the pipeline checks the manifest by image_uid. A valid existing JPEG with a matching stored checksum is skipped.

Missing or corrupt files are downloaded again through a temporary .partfile and moved atomically only after JPEG validation. Repeating the same command therefore does not create duplicate images.

Testing
Run the offline test suite:

bash
python -m pytest -q
Tests mock network boundaries and use generated JPEG fixtures.

Scaling limitations
The API backend is intended for pilots and moderate batches. --images-per-speciesoperates on a bounded candidate set and does not guarantee exhaustive coverage of every species in a large order.

For larger requests:

Run a dry run first.

Pace downloads conservatively.

Use the iNaturalist Open Dataset bulk metadata/object-storage workflow for exhaustive or whole-dataset processing.

Troubleshooting
bird-image-downloadis not found
Run pipx ensurepath, restart the terminal, and use pipx listto confirm that the application is installed.

Git is missing
Install Git using the GitHub installation guide , then retry the pipxinstallation.

Python is unsupported
Install Python 3.11 or newer from python.org and ensure that pipxselects that version.

Linux reports an externally managed environment
Install and use pipxthrough the distribution package instead of installing the application into system Python. This avoids the PEP 668 externally managed-environment restriction.

An installed version needs updating or repair
bash
pipx upgrade bird-image-pipeline
If the upgrade fails:

bash
pipx uninstall bird-image-pipeline
pipx install git+https://github.com/chpngyu/open-bird-image-pipeline.git
No count was supplied
Add --limit N, use --images-per-species N, or run the command interactively.

An order cannot be resolved
Use a current scientific order name such as Passeriformes.

No eligible images were found
Returned observations may lack CC0 or CC BY photographs on the open-data host.

HTTP throttling or server errors occur
Rerun later. Transient API responses are retried with backoff.

An existing image is corrupt
Rerun the same command. The corrupt target is replaced after the replacement passes validation.

Metadata is sparse
Observation descriptions, annotations, dates, and public place text are optional on iNaturalist.

Diff-AE bird training
The optional Diffusion Autoencoder workflow consumes manual decisions from inaturalist_images/image_review.csv. It selects only rows where accept == 1, compares recorded dimensions with decoded files, and writes centered 500 × 500 RGBA PNGs. Source images are never overwritten.

Diff-AE is an external, one-time installation and is not installed by the pipeline. Clone the upstream Diff-AE repository , install its optional dependencies in a CUDA-enabled PyTorch environment, and replace:

Python
from numpy.lib.function_base import flip
with:

Python
from numpy import flip
in experiment.pyfor NumPy 2 compatibility.

Set DIFFAE_ROOTor pass --diffae-root:

bash
conda activate torch_gpu
python -m pip install -e ".[diffae,test]"

# POSIX shell
export DIFFAE_ROOT=/path/to/diffae

# Windows PowerShell
$env:DIFFAE_ROOT = "C:\path\to\diffae"
Run the workflow stages:

bash
python scripts/run_diffae_birds.py preflight
python scripts/run_diffae_birds.py prepare
python scripts/run_diffae_birds.py cross-validate --resume
python scripts/run_diffae_birds.py train-final --resume
python scripts/run_diffae_birds.py embed
python scripts/run_diffae_birds.py visualize
python scripts/run_diffae_birds.py report
Run every stage in order with:

bash
python scripts/run_diffae_birds.py all
--smokeis only an integration check and does not establish model quality. Prepared images remain 500 × 500; model tensors are 256 × 256 with batch size 1 and gradient accumulation 16. The detected 2 GiB MX550 may require changing image_sizeto 128 after an out-of-memory smoke run.

Diff-AE source files may have different dimensions because its loader resizes and center-crops them. Batches receive identical tensor shapes because this workflow uses uniform 256 × 256 model inputs. The semantic latent width is Diff-AE's style_ch; latent_dimconfigures it consistently. The current test configuration uses 64 latent dimensions and 50 epochs.

Outputs below artifacts/diffae/are ignored by Git. They include quality-control and fold manifests, summaries and checkpoints, latent matrices, largest-variance and PCA coordinate CSVs/plots, logs, JSON state, and a Markdown report.

Training writes loss.csvand loss.pngbeside each final or fold checkpoint.

After embedding and visualization, launch the interactive Streamlit viewer:

bash
python -m streamlit run scripts/streamlit_diffae_latents.py -- \
  --artifact-dir artifacts/diffae/birds-500-latent64
The viewer supports Plotly point inspection, prepared-image previews, metadata selection, filename lookup, and final-training loss history.

Project status
This project is designed for reproducible, attribution-preserving image collection and moderate-scale experimentation. Review the generated metadata and licensing fields before publishing or redistributing any derived dataset.
