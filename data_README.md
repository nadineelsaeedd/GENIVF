# Data

The dataset used in this study is **not included in this repository** because it is a third-party dataset with its own licence and deposit.

## Download instructions

1. Go to the Mendeley Data page:  
   👉 **https://doi.org/10.17632/x5s3ky9cpv.1**

2. Download the file: `Embryo dataset_2542.xlsx`

3. Place it in this folder:
   ```
   data/Embryo dataset_2542.xlsx
   ```

The notebook will read it directly from this path.

## Dataset description

- **Source:** Gansu Provincial Maternity and Child-care Hospital, China (January 2016 – August 2020)
- **Depositor:** Wang M. (2026)
- **Records:** 2,542 rows, 38 columns (Chinese headers — translated in the notebook)
- **Licence:** CC BY 4.0
- **Unit of analysis:** Embryo records nested in patients (1,415 unique patients after deduplication — see paper Section 3.2)
- **Outcome variable:** Clinical pregnancy (binary)

## Important note on data quality

Before using this dataset for any modelling, please read the data audit in our paper. The file contains outcome-specific imputed constants in 13 columns that inflate ML model performance by up to 0.237 AUROC points. Our notebook corrects this before training any model.
