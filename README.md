# Seven-Segment OCR: TrOCR vs PaddleOCR

A comparative evaluation of Transformer-based and PaddleOCR-based text recognition approaches on seven-segment display images.

## Overview

This project evaluates OCR models on seven-segment display images containing numeric readings, including decimal values. The objective is to compare recognition performance and inference latency and investigate whether image preprocessing and inference optimizations improve results.

Two approaches were evaluated:

- **TrOCR:** Transformer-based OCR using the pretrained `microsoft/trocr-base-printed` checkpoint.
- **PaddleOCR + DBNet:** PaddleOCR's detection-and-recognition pipeline, evaluated on the available cropped display images.

The experiments include input resizing, CLAHE contrast enhancement, FP16 inference, and key-value cache configuration where applicable.

## Repository Structure

```text
seven-segment-ocr-benchmark/
├── README.md
├── TrOCR_Seven_Segment_Evaluation.ipynb
└── PaddleOCR_DBNet_Evaluation.ipynb
```

*The notebook filenames above are suggested names. Rename them to match the files you upload.*

## Dataset

- **Source:** [Seven-Segment OCR Dataset](https://github.com/renjithsasidharan/seven-segment-ocr)
- **Image dimensions:** 200 × 31 pixels in the original dataset.
- **Evaluation set:** 48 images.
- **Ground truth:** Text labels provided with the dataset.

The evaluation uses the same 48 test images for both approaches. Text normalization removes whitespace and case-insensitive `kWh` text while preserving decimal points and leading zeros.

## Experimental Setup

| Component | Details |
|---|---|
| TrOCR checkpoint | `microsoft/trocr-base-printed` |
| PaddleOCR | PP-OCRv5 configuration |
| TrOCR inference hardware | NVIDIA Tesla T4 GPU |
| TrOCR precision | FP32 and FP16 |
| Preprocessing experiments | 2× resizing and CLAHE |
| Evaluation metrics | Exact-match accuracy, CER, WER, mean latency, P50 and P99 |
| Test set size | 48 images |

The pretrained TrOCR checkpoint was evaluated without task-specific fine-tuning in these experiments.

## Results

### PaddleOCR Results

| Configuration | Exact Match Accuracy | CER | WER | Mean Latency |
|---|---:|---:|---:|---:|
| Original 200 × 31 | 22.92% | 46.48% | 77.08% | 31.98 ms |
| Resized 800 × 124 | 6.25% | 58.80% | 93.75% | 52.06 ms |

### TrOCR Results

| Configuration | Exact Match Accuracy | CER | WER | Mean Latency |
|---|---:|---:|---:|---:|
| FP32 baseline | 14.58% | 37.32% | 85.42% | 98.10 ms |
| FP32 + 2× resize | 14.58% | 35.56% | 85.42% | 100.19 ms |
| FP32 + CLAHE | 10.42% | 34.51% | 89.58% | 105.73 ms |
| FP16 | 14.58% | 37.32% | 85.42% | 80.61 ms |
| FP16 + KV cache | 14.58% | 37.32% | 85.42% | 85.28 ms |
| FP16 + 2× resize | 14.58% | 35.56% | 85.42% | 76.36 ms |

### Key Observations

- Original-resolution PaddleOCR achieved the highest exact-match accuracy in this evaluation: **22.92%**.
- TrOCR with FP16 reduced mean generation latency from 98.10 ms to 80.61 ms without changing its measured recognition metrics.
- TrOCR with FP16 and 2× resizing achieved a mean generation latency of 76.36 ms and a CER of 35.56%.
- CLAHE produced the lowest observed TrOCR CER (34.51%), but exact-match accuracy decreased to 10.42%.
- Resizing PaddleOCR inputs to 800 × 124 reduced exact-match accuracy and increased CER in this experiment.

## Metrics Explained

- **Exact-match accuracy:** Percentage of images whose complete normalized prediction exactly matches the ground-truth label.
- **Character Error Rate (CER):** Character-level edit errors relative to the ground-truth text. Lower is better.
- **Word Error Rate (WER):** Sequence-level word error metric. For short numeric readings, interpret this alongside exact-match accuracy and CER.
- **Mean latency:** Average measured inference time.
- **P50 latency:** Median measured inference time.
- **P99 latency:** Latency below which 99% of measured inference times fall.

## Limitations

- The evaluation contains only 48 test images; results may not generalize to other displays or imaging conditions.
- The results are exploratory and do not establish that either model is universally superior.
- Multiple configurations were evaluated on the same test set, so a separate validation set and an untouched test set would provide a stronger final comparison.
- TrOCR latency measures model generation after image preprocessing. PaddleOCR latency measures a broader prediction pipeline. These latency measurements are not directly equivalent.
- PaddleOCR includes detection and recognition components, but the inputs are already cropped display strips. This experiment does not independently establish DBNet's detection accuracy.
- The low exact-match accuracy indicates that further adaptation and evaluation are needed before reliable deployment.

## Reproducibility

1. Clone or download the source dataset.
2. Open the relevant notebook in Google Colab or a compatible Python environment.
3. Install the dependencies listed in the notebook.
4. Run the cells in order to reproduce preprocessing, inference, and evaluation.
5. Compare predictions with ground-truth labels and inspect the reported metrics.

GPU availability and package versions can affect latency and reproducibility.

## Future Work

- Fine-tune OCR models specifically for seven-segment displays.
- Evaluate on a larger, independent test set.
- Investigate character-level errors involving digits, decimal points, and leading zeros.
- Measure end-to-end latency using consistent timing boundaries.
- Evaluate detection and recognition separately when using DBNet.

## Acknowledgements

- Dataset: [renjithsasidharan/seven-segment-ocr](https://github.com/renjithsasidharan/seven-segment-ocr)
- TrOCR pretrained checkpoint: [microsoft/trocr-base-printed](https://huggingface.co/microsoft/trocr-base-printed)
- OCR framework: [PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR)

## License

The notebooks and original dataset may have different licensing terms. Check the source dataset repository and the model/framework licenses before redistributing dataset images or other third-party materials.
