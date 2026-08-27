# Elliptic dataset

## Source

Use the original Elliptic Data Set published through Kaggle:

https://www.kaggle.com/datasets/ellipticco/elliptic-data-set

Review and comply with the license and dataset terms shown on the source page.

## Required files

After downloading and extracting the dataset, this project expects:

data/raw/elliptic_bitcoin_dataset/elliptic_txs_features.csv
data/raw/elliptic_bitcoin_dataset/elliptic_txs_classes.csv
data/raw/elliptic_bitcoin_dataset/elliptic_txs_edgelist.csv

Do not rename these files.

## Manual download

1. Sign in to Kaggle.
2. Open the dataset page.
3. Download the dataset archive.
4. Extract it under `data/raw/`.
5. Confirm that the three CSV files have the exact paths listed above.

Avoid creating an extra nested directory such as:

data/raw/elliptic_bitcoin_dataset/elliptic_bitcoin_dataset/

## Kaggle command-line alternative

Install the project dependencies, configure Kaggle credentials according to
Kaggle's documentation, and run from the project root:

kaggle datasets download \
  -d ellipticco/elliptic-data-set \
  -p data/raw \
  --unzip

Inspect the extracted directories afterward and ensure the required files end
up in `data/raw/elliptic_bitcoin_dataset/`.

Never commit Kaggle credentials or dataset files.

## Dataset verification

Run the setup cell at the beginning of `notebooks/main.ipynb`. It reports every
missing file and stops before the analysis continues.