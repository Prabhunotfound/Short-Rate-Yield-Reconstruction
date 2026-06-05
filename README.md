# Short Rate Yield Reconstruction

A pipeline for reconstructing the full US zero-coupon yield curve from the 3-month rate alone, using a calibrated **CIR++ model with a Jump-Diffusion extension**, tested against the 2022–2024 interest rate hiking cycle.

---

## Setup Instructions

### 1. Cloning the Repo

```bash
git clone https://github.com/Prabhunotfound/Short-Rate-Yield-Reconstruction
cd Short-Rate-Yield-Reconstruction
```

### 2. Environment Creation

#### Using `venv` (pip)

**Linux/macOS**
```bash
python -m venv venv
source venv/bin/activate
```

**Windows**
```bash
python -m venv venv
venv\Scripts\activate
```

#### Using `conda`
```bash
conda create -n rate-model python=3.11
conda activate rate-model
```

### 3. Install Dependencies

```bash
pip install numpy pandas matplotlib seaborn scipy gdown os
```

---

## Project Structure

```
├── Project.ipynb               # Full pipeline 
├── train_data.csv              # Daily zero-coupon yields, 9 maturities (training)
├── test_data.csv               # Out-of-sample yield data, all maturities
├── test_data_3M.csv            # Test set with 3M rate only (model input)
├── plots/                      # Figures auto-saved during notebook execution
└── README.md
```

---

## Dataset

The three CSV files contain daily **US zero-coupon bond yields** across 9 maturities (`3M · 6M · 9M · 1Y · 2Y · 5Y · 10Y · 20Y · 30Y`), sourced from the drive link provided in the notebook's data loading cell.

Download and place them in the root directory before running the notebook.

> **Note:** The notebook was originally run on **Google Colab**. Update the file paths at the top of the data loading cell if running locally ( Just have to uncomment out the gdown.download() lines ).

---

## Colab Link

The following is the link to the Google Colab File : 
[Google Colab](https://colab.research.google.com/drive/1kbeTPe4FXFZGESrfANmJi56Jn_S3FfeE?usp=sharing)
