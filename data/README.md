# Data

This project uses the **IEEE-CIS Fraud Detection** dataset from Kaggle.

Due to file size (~1.3 GB), the data files are not included in this repository.

## How to download

1. Go to https://www.kaggle.com/c/ieee-fraud-detection/data
2. Accept the competition rules
3. Download `train_transaction.csv` and `train_identity.csv`
4. Place both files in this `data/` directory

## Alternative (Kaggle CLI)

```bash
pip install kaggle
kaggle competitions download -c ieee-fraud-detection -p data/
unzip data/ieee-fraud-detection.zip -d data/
```

## Files expected

```
data/
    train_transaction.csv   (~690 MB, 590,540 rows x 394 columns)
    train_identity.csv      (~30 MB, 144,233 rows x 41 columns)
```
