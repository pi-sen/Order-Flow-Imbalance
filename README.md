# Order Flow Imbalance (OFI) Construction

## Objective

This program implements Order Flow Imbalance (OFI) features from high-frequency limit order book (LOB) data and evaluates their application in price impact modeling and cross-asset relationships. OFI is a microstructure-based feature that captures the net imbalance between supply and demand, considered superior to trade volume for modeling short-term returns. The implementation is based on methodologies presented in "Cross-impact of order flow imbalance in equity markets" by Rama Cont, Mihai Cucuringu & Chao Zhang.

## Code Structure & Flow

### 🔹 1. Imports and Configuration
* Standard imports: pandas, numpy, sklearn, and matplotlib
* Column suffixes for bid/ask prices and sizes (bid_px_00, ask_sz_05, etc.) defined for the top 10 levels of the order book

### 🔹 2. Data Loading and Preprocessing
* Primary dataset loaded via pd.read_csv()
* Timestamps converted using pd.to_datetime(), and dataframe sorted by ts_event to preserve chronological order
* Depth column names programmatically generated using list comprehensions

### 🔹 3. Best-Level OFI Calculation
**Function**: compute_best_ofi(row_t, row_tm1)
* Compares bid/ask prices and sizes at level 0 (best bid and best ask) between two time points
* Implements logic from research paper (Cont et al.): if price increases, the new quantity is added; if it stays the same, the change in size is used; if it decreases, the previous size is subtracted
* Applied across the dataset to produce OFI_best

### 🔹 4. Multi-Level OFI Calculation
**Function**: compute_multi_level_ofi(row_t, row_tm1)
* Loops through LOB levels 0 to 9 and applies the same logic as best-level OFI for each depth
* Aggregates across all levels to generate a single OFI_multi value per timestamp

### 🔹 5. Integrated OFI Calculation via PCA
* Constructs a list of 10-dimensional OFI vectors at each timestamp
* Applies Principal Component Analysis (PCA) using sklearn to extract the first principal component
* The first component serves as the Integrated OFI, capturing the most informative combination of depth-level OFIs

### 🔹 6. Synthetic Cross-Asset Dataset
* Creates a synthetic dataset by duplicating the AAPL data and assigning fake tickers (AAPL, GOOG, MSFT)
* Each "synthetic" asset is given slightly perturbed values to simulate distinct order book behavior
* A new symbol column is added to facilitate group-wise calculations

### 🔹 7. Cross-Asset OFI Calculation
**Logic**:
* For each asset, sum the OFI values of all other assets at each timestamp
* This represents a naive cross-asset pressure signal that approximates the influence of external order flow on the asset in question
* Note: This is a simplification of the cross-impact model described in the paper, which uses Lasso regression to formally estimate pairwise impact coefficients

### 🔹 8. Output
* Final features (OFI_best, OFI_multi, OFI_integrated, OFI_cross) are stored in the dataframe for export or further analysis

## Testing Cross-Asset OFI Using Synthetic Data

Since real multi-asset LOB data was unavailable, we:
* Replicated AAPL data into multiple "assets"
* Applied noise or transformations to simulate realistic asset-specific behavior
* Evaluated how well cross-asset OFI aligns with or diverges from asset-specific OFIs

This approach allows testing the cross-impact logic without needing real cross-asset LOB feeds.

## Summary of Outputs

| Feature | Description |
|---------|-------------|
| OFI_best | Order flow imbalance at the best (top) LOB level |
| OFI_multi | Aggregated OFI from top 10 levels of LOB |
| OFI_integrated | PCA-based integrated OFI signal |
| OFI_cross | Sum of OFIs from other synthetic assets at timestamp |
