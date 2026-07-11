# Complete-Triplet CNN-BiLSTM for Automated Sleep Staging in Chronic Pain Cohorts

Code, trained models, and experiment logs for a biologically continuous sequence-modeling
approach to automated sleep stage classification, evaluated on a mixed healthy-control /
Zoster-Associated Neuralgia (ZAN) polysomnography cohort (OpenNeuro **ds007181**).

The model combines a multi-resolution 1D-CNN front end with a Bidirectional LSTM and
self-attention classification head, trained on **complete, non-padded 90-second triplet
windows** (three consecutive 30-second epochs) to avoid the zero-padding artifacts common
in standard sleep-staging pipelines. The project also includes six reimplemented baseline
architectures trained under identical conditions for a controlled comparison, and a full
dual-split validation protocol (epoch-level and subject-level/patient-independent) to
report both the conventional and the leakage-free generalization numbers.

---

## Highlights

- **Complete-Triplet framing** — no synthetic zero-padding at recording boundaries; only
  epochs with a full, continuous past/present/future context are used.
- **Dual-split evaluation** — results reported under both a conventional epoch-level split
  and a strict subject-level (patient-independent) holdout, so the leakage gap is visible
  rather than hidden.
- **Six reimplemented baselines** (DeepSleepNet, SeqSleepNet, U-Time, SleepEEGNet,
  AttnSleep, LightSleepNet) trained on identical data, preprocessing, and splits for a
  fair, controlled comparison — not just citations to numbers from other datasets.
- **Explainable AI** — channel-occlusion and frequency-band-occlusion sensitivity analysis
  per sleep stage, for both validation splits.
- **Edge-oriented** — ~198K trainable parameters, with a theoretical FLOPs analysis for
  microcontroller-class deployment.

---

## Results summary

| Split | Accuracy | Macro F1 |
|---|---|---|
| Epoch-level | 85.60% | 0.83 |
| Subject-level (patient-independent) | 72.52% | 0.69 |

| Model | Params | Epoch-Level Acc. | Subject-Level Acc. |
|---|---|---|---|
| DeepSleepNet | ~22.5M | 82.30% | 72.36% |
| SeqSleepNet | ~1.2M | 82.11% | 71.88% |
| U-Time | ~2.1M | 82.29% | 71.34% |
| SleepEEGNet | ~1.5M | 83.22% | 72.37% |
| AttnSleep | ~1.3M | 79.42% | 70.09% |
| LightSleepNet | ~750K | 80.76% | 68.32% |
| **Proposed Model** | **197,829** | **85.60%** | **72.52%** |

Full per-class metrics, confusion matrices, ROC curves, and XAI figures for both splits
are in [`results/`](./results).

---

## Repository structure

```
.
├── notebooks/
│   ├── 01_data_preprocessing.ipynb        # Triplet construction, instance normalization
│   ├── 02_train_proposed_model.ipynb      # Complete-Triplet CNN-BiLSTM training (both splits)
│   ├── 03_train_baselines.ipynb           # 6 reimplemented baseline models (both splits)
│   ├── 04_evaluation_and_metrics.ipynb    # Classification reports, confusion matrices, ROC
│   └── 05_explainable_ai.ipynb            # Channel/band occlusion sensitivity analysis
├── src/
│   ├── data/
│   │   ├── triplet_generator.py           # CompleteTripletGenerator (Keras Sequence)
│   │   └── splits.py                      # Epoch-level & subject-level split logic
│   ├── models/
│   │   ├── proposed_model.py              # Multi-resolution CNN-BiLSTM-Attention
│   │   ├── deepsleepnet.py
│   │   ├── seqsleepnet.py
│   │   ├── utime.py
│   │   ├── sleepeegnet.py
│   │   ├── attnsleep.py
│   │   └── lightsleepnet.py
│   └── xai/
│       ├── channel_occlusion.py
│       └── band_occlusion.py
├── results/
│   ├── epoch_level/                       # Confusion matrix, ROC, XAI figures (epoch split)
│   ├── subject_level/                     # Confusion matrix, ROC, XAI figures (subject split)
│   └── model_comparison_table.csv         # All 7 models × both splits
├── checkpoints/                           # Trained model weights (.keras) — see note below
├── paper/
│   └── Biologically_Continuous_Sequence_Modeling_...pdf
├── requirements.txt
└── README.md
```

> **Note on checkpoints:** trained `.keras` model files can be large. If they exceed
> GitHub's file size limits, use [Git LFS](https://git-lfs.github.com/) or host them
> separately (e.g., Hugging Face Hub, Zenodo) and link them here instead of committing
> the binaries directly.

---

## Dataset

This project uses the **ds007181** dataset from OpenNeuro:

> Li Li, Qi Han, Haolei Bai, Xiaolong Zhang, Yong Liu, and Chao He (2026).
> *Structural MRI, Resting-state fMRI, and PSG/EEG Dataset of Zoster-associated Neuralgia.*
> OpenNeuro. https://doi.org/10.18112/openneuro.ds007181.v1.0.1

The PSG/EEG subset contains overnight polysomnography recordings from healthy control
(HC) and Zoster-Associated Neuralgia (ZAN) subjects, manually scored per AASM criteria
(Wake, N1, N2, N3, REM). After quality filtering, this project uses recordings from
**52 subjects (28 HC, 24 ZAN)**.

The dataset is **not included in this repository**. Download it directly from
[OpenNeuro](https://openneuro.org/datasets/ds007181) and update the data paths in
`src/data/` before running the notebooks.

---

## Setup

```bash
git clone https://github.com/<your-username>/complete-triplet-bilstm-sleep-staging.git
cd complete-triplet-bilstm-sleep-staging
pip install -r requirements.txt
```

Tested with Python 3.10+, TensorFlow/Keras 2.x, on Google Colab (NVIDIA Tensor Core GPU).

### Minimal `requirements.txt`
```
tensorflow>=2.15
numpy
pandas
scikit-learn
scipy
seaborn
matplotlib
tqdm
```

---

## Reproducing the results

1. Download ds007181 and run `notebooks/01_data_preprocessing.ipynb` to build the
   complete-triplet index and both (epoch-level, subject-level) train/val splits.
2. Run `notebooks/02_train_proposed_model.ipynb` to train the proposed model under both
   splits. Checkpoints and per-split classification reports are saved automatically.
3. Run `notebooks/03_train_baselines.ipynb` to train all six baseline architectures
   under identical conditions.
4. Run `notebooks/04_evaluation_and_metrics.ipynb` to regenerate confusion matrices, ROC
   curves, and the full model-comparison table.
5. Run `notebooks/05_explainable_ai.ipynb` for channel- and band-occlusion sensitivity
   analysis per sleep stage, for both splits.

---

## Citation

If you use this code or the accompanying results, please cite:

```bibtex
@article{shah2026biologically,
  title   = {Biologically Continuous Sequence Modeling for Automated Sleep Staging in
             Chronic Pain Cohorts using Complete-Triplet BiLSTMs},
  author  = {Shah, Suryansh and Kose, Mangesh},
  year    = {2026},
  note    = {Manuscript}
}
```

Also cite the dataset (see [Dataset](#dataset) section above).

---

## License

Specify a license here (e.g., MIT, Apache 2.0) — add a `LICENSE` file to the repo root.
Note that any use of ds007181 itself is separately governed by its **CC0** license.

---

## Acknowledgments

Department of Artificial Intelligence and Machine Learning, Manipal University Jaipur,
and Indian Institute of Information Technology, Nagpur (IIITN).
