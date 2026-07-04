# Fruit Vision

A practice project for classifying fresh and rotten fruit images (apples, bananas, oranges) using the **YOLO26n-cls** model from [Ultralytics](https://github.com/ultralytics/ultralytics).

The model distinguishes between 6 classes:
- `freshapples`
- `freshbanana`
- `freshoranges`
- `rottenapples`
- `rottenbanana`
- `rottenoranges`

---

## Dataset

This project uses the following public dataset from Kaggle:

**Fruits fresh and rotten for classification**
https://www.kaggle.com/sriramr/fruits-fresh-and-rotten-for-classification

### Preparing the dataset
1. Download the dataset from the link above (a Kaggle account is required).
2. Extract the zip file.
3. Arrange the folder structure inside the project root as follows:

```
Fruit-vision/
└── datasets/
    ├── train/
    │   ├── freshapples/
    │   ├── freshbanana/
    │   ├── freshoranges/
    │   ├── rottenapples/
    │   ├── rottenbanana/
    │   └── rottenoranges/
    └── test/
        ├── freshapples/
        ├── freshbanana/
        ├── freshoranges/
        ├── rottenapples/
        ├── rottenbanana/
        └── rottenoranges/
```

Note: there is no separate `val` folder in this dataset. In its absence, Ultralytics automatically falls back to using the `test` folder as val (see the "Known Limitations" section below).

---

## Installation and Setup

```bash
# Create a virtual environment (optional but recommended)
python -m venv venv
venv\Scripts\activate      # Windows
# source venv/bin/activate # Linux / macOS

# Install required packages
pip install ultralytics torch
```

The base weights `yolo26n-cls.pt` are downloaded automatically by Ultralytics on first run.

---

## Running the Project

Open `index.ipynb` in Jupyter (or JupyterLab) and run the cells in order. The notebook performs the following: fine-tunes the model, evaluates it, tests it on a few sample images, and exports the final model.

```bash
jupyter notebook index.ipynb
```

### Training configuration
| Parameter | Value |
|---|---|
| Base model | `yolo26n-cls.pt` |
| Epochs | 50 |
| Image size | 224x224 |
| Batch size | 16 |
| Device | CPU |

### Training time
Training this model on **CPU** (Intel Core i7-6500U) for 50 epochs took approximately **14 hours**. If you have a GPU available, changing `device="cpu"` to `device="0"` (or your GPU index) will drastically reduce this time.

---

## Results

After training, the model reached the following accuracy on the evaluation data:

- **Top-1 Accuracy:** 1.0000
- **Top-5 Accuracy:** 1.0000

This number should be read with caution, for reasons explained below.

---

## Known Limitations and Issues

This project is purely for practice, and the reported 100% accuracy is **not a reliable measure of the model's real-world performance**. The reasons are:

1. **No independent `val` set:** Since the dataset only has `train` and `test` folders, Ultralytics automatically used the `test` folder both for selecting the best checkpoint during training (`best.pt`) and for the final evaluation. This means the final model was selected based on the exact same data that was supposed to be held out purely for final testing.

2. **Data leakage within the dataset itself:** In the original Kaggle dataset, base images were multiplied by rotating them at various angles (`rotated_by_0`, `rotated_by_15`, ...) and then split into `train` and `test`. This split appears to have been done *after* the rotation step rather than before, so rotated versions of the same base image may appear in both `train` and `test`. As a result, the model likely already saw a nearly identical version (just at a different angle) of test images during training.

Together, these two issues make the reported accuracy unrealistically optimistic. To properly measure the model's real performance, one should:
- Build a separate `val` set split by base image (before rotation), or
- Test the model on images completely outside this dataset (e.g., photos taken with a phone).

---

## Exported Model

After training, the best weights (`best.pt`) were exported to **ONNX** (`.onnx`) format, for deployment in lightweight environments and various frameworks such as OpenCV DNN, ONNX Runtime, etc.

The exported file is located alongside the original model weights:
```
runs/classify/Fruit-Vision/yolo26_fruit_run/weights/
├── best.pt
├── best.onnx
└── last.pt
```

---

## Project Structure

```
Fruit-vision/
├── index.ipynb                   # Main notebook (training/evaluation/export)
├── yolo26n-cls.pt                 # Base pretrained weights
├── datasets/
│   ├── train/
│   │   ├── freshapples/
│   │   ├── freshbanana/
│   │   ├── freshoranges/
│   │   ├── rottenapples/
│   │   ├── rottenbanana/
│   │   └── rottenoranges/
│   └── test/
│       ├── freshapples/
│       ├── freshbanana/
│       ├── freshoranges/
│       ├── rottenapples/
│       ├── rottenbanana/
│       └── rottenoranges/
└── runs/
    └── classify/
        ├── Fruit-Vision/
        │   └── yolo26_fruit_run/
        │       ├── args.yaml
        │       ├── results.csv
        │       ├── train_batch*.jpg      # Sample training batches
        │       ├── val_batch*_labels.jpg  # Ground-truth labels on val batches
        │       ├── val_batch*_pred.jpg    # Model predictions on val batches
        │       └── weights/
        │           ├── best.pt
        │           ├── best.onnx
        │           └── last.pt
        └── val/                            # Output of the standalone val run
```

---

## Final Note

This project was built purely to practice working with Ultralytics YOLO for a classification task and is not production-ready, particularly due to the data leakage issue described above.