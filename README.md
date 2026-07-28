<div align="center">

# 🩺 CUE-SFDA

### Consistency-guided Uncertainty Estimation for Source-Free Medical Image Segmentation

**A research implementation for source-free adaptation in medical image segmentation**

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-1.12%2B-EE4C2C?logo=pytorch&logoColor=white)
![Task](https://img.shields.io/badge/Task-Binary%20Segmentation-5B8FF9)
![Code](https://img.shields.io/badge/Code-Research%20Release-7A5AF8)

🔒 **Source-free** · 🎯 **Uncertainty-guided** · 🧠 **Medical segmentation** · ♻️ **Reusable predictions**

</div>

> [!NOTE]
> The manuscript has not yet been formally published. Publication information
> and the final citation will be added when available.

CUE-SFDA adapts a source-trained binary segmentation model to an unlabeled
target domain. This release contains the target-domain adaptation and test
code. It does not include medical images, pretrained checkpoints, generated
pseudo labels, or experiment outputs.

## 📦 Repository structure

```text
CUE-SFDA-release/
|-- configs/
|   `-- train_example.json
|-- cue_sfda/
|   |-- __init__.py
|   `-- core.py
|-- networks/
|   |-- backbone/
|   |-- sync_batchnorm/
|   |-- aspp.py
|   |-- decoder.py
|   `-- deeplabv3.py
|-- train.py
|-- test.py
|-- requirements.txt
|-- .gitignore
`-- README.md
```

Only files required by training and evaluation are included. Personal paths,
datasets, model weights, cached predictions, ablation scripts, visualizations,
and intermediate experiment outputs are excluded.

## ⚙️ Installation

Python 3.10 or newer and a CUDA-capable PyTorch installation are recommended.

```bash
git clone <repository-url>
cd CUE-SFDA-release

python -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

Install a PyTorch build matching your CUDA version if the default pip package
is not suitable for your machine.

## 🗂️ Data layout

The same directory convention is used for CFP, endoscopy polyp, and prostate
datasets:

```text
data/
`-- TargetDomain/
    |-- train/
    |   |-- image/
    |   |   `-- sample.png
    |   `-- mask/              # not read during source-free adaptation
    `-- test/
        |-- image/
        |   `-- sample.png
        `-- mask/
            `-- sample.png
```

For CFP, `train_split` and `test_split` may be set to `train/ROIs` and
`test/ROIs`, while `kind` is `od` or `oc`. For polyp and prostate data, the
usual values are `train`, `test`, and `mask`.

All input images are converted to RGB and resized to 512 x 512 by default.
Masks must be binary PNG files with the same filenames as their images.

## 📥 Required inputs

### Source checkpoint

`model_file` must be a complete source-trained MobileNet-DeepLabV3+ checkpoint.
Both of the following checkpoint layouts are accepted:

```python
{"model_state_dict": model.state_dict()}
```

or a state dictionary saved directly with `torch.save(model.state_dict(), ...)`.

The legacy MobileNet ImageNet download is disabled because adaptation restores
the complete source checkpoint before target-domain inference.

### Affinity-refined pseudo labels

High-quality target samples use affinity-refined pseudo labels. Set
`affinity_dir` to a directory containing one binary PNG for every target
training image:

```text
affinity/
`-- TargetDomain/
    `-- mask/
        `-- sample.png
```

The filenames must exactly match `train/image`. These labels are preprocessing
artifacts and are not source ground truth. The training entry point validates
their completeness before starting GPU work.

## 🚀 Training

Copy and edit the example configuration:

```bash
cp configs/train_example.json configs/train_local.json
```

Important fields:

- `data_dir`: dataset root;
- `target`: target-domain folder;
- `model_file`: source-domain checkpoint;
- `affinity_dir`: affinity-refined target pseudo labels;
- `output_root`: experiment output directory;
- `k`, `tau`, `gamma`: CUE-SFDA parameters;
- `device`: for example `cuda:0`.

Run the complete adaptation pipeline:

```bash
python train.py --config configs/train_local.json --stage all
```

The stages may also be run separately:

```bash
python train.py --config configs/train_local.json --stage cache
python train.py --config configs/train_local.json --stage partition
python train.py --config configs/train_local.json --stage train
```

Existing results are reused. Regeneration must be requested explicitly:

```bash
python train.py --config configs/train_local.json \
  --stage all \
  --force-cache \
  --force-partition \
  --force-train
```

The final checkpoint is written below:

```text
outputs/<run-name>/training/K10_tau0p800_gamma0p800/final_model.pth.tar
```

The same directory also contains:

- `metrics.json`;
- `per_image_metrics.csv`;
- `training_history.json`.

Prediction caches and partition files are retained under the run directory to
support reproducibility.

## 🧪 Evaluation

Evaluate a trained checkpoint:

```bash
python test.py \
  --checkpoint outputs/example_target/training/K10_tau0p800_gamma0p800/final_model.pth.tar \
  --data-root data \
  --target TargetDomain \
  --test-split test \
  --kind mask \
  --output-dir test_results/example_target \
  --device cuda:0 \
  --save-predictions
```

Outputs:

```text
test_results/example_target/
|-- metrics.json
|-- per_image_metrics.csv
`-- predictions/              # when --save-predictions is supplied
```

Reported metrics are computed only from the test images and test masks.

## ♻️ Reproducibility

Keep the following files when publishing experiment results:

- the source checkpoint and its SHA-256;
- the resolved training configuration;
- `prediction_cache/manifest.json`;
- all arrays under `prediction_cache/`;
- the high/low-quality partition;
- the final checkpoint and per-image metrics.

The cache manifest records augmentation seeds, MixUp partners and coefficients,
the source-checkpoint SHA-256, and the fixed view permutation used for every K.

## ✅ Public release checklist

Before publishing the repository:

1. choose and add an open-source `LICENSE`;
2. add the final paper citation, DOI, and author contact;
3. publish or document access to the allowed datasets and source checkpoints;
4. verify that redistributed pseudo labels comply with each dataset license;
5. remove any private paths or credentials from additional files.

## 📝 Citation

The manuscript has not yet been formally published. For the moment, please
refer to the project using its title:

> Consistency-guided Uncertainty Estimation for Source-Free Medical Image
> Segmentation.

The complete BibTeX entry, author list, venue, and DOI will be added after the
publication metadata becomes available.
