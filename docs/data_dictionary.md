"""ETL pipeline for BigBasket Product Pricing & Category Analysis.
NST DVA Capstone 2 — BigBasket Market Analysis project.

This script replicates the full cleaning & feature-engineering pipeline from
notebooks/02_cleaning.ipynb as a standalone, reusable Python module.

Usage
-----
python scripts/etl_pipeline.py \
    --input  data/raw/BigBasket_Products.csv \
    --output data/processed/bigbasket_cleaned.csv
"""

from __future__ import annotations

import argparse
import warnings
from pathlib import Path

import numpy as np
import pandas as pd

warnings.filterwarnings("ignore")

# ---------------------------------------------------------------------------
# Column-name helpers
# ---------------------------------------------------------------------------

def normalize_columns(df: pd.DataFrame) -> pd.DataFrame:
    """Convert column names to clean snake_case."""
    cleaned = (
        df.columns.str.strip()
        .str.lower()
        .str.replace(r"[^a-z0-9]+", "_", regex=True)
        .str.strip("_")
    )
    result = df.copy()
    result.columns = cleaned
    return result


# ---------------------------------------------------------------------------
# Core cleaning steps
# ---------------------------------------------------------------------------

def drop_non_analytical(df: pd.DataFrame) -> pd.DataFrame:
    """Drop columns that carry no analytical value (e.g. row-index columns)."""
    cols_to_drop = [c for c in ["index"] if c in df.columns]
    if cols_to_drop:
        df = df.drop(columns=cols_to_drop)
        print(f"Dropped columns: {cols_to_drop}")
    return df


def handle_missing_values(df: pd.DataFrame) -> pd.DataFrame:
    """Impute or fill all missing values according to the documented strategy."""
    df["product"] = df["product"].fillna("Unknown Product")
    df["brand"] = df["brand"].fillna("Unknown")
    df["description"] = df["description"].fillna("No description available")

    # Impute rating with category-wise median, then global median as fallback
    df["rating"] = df.groupby("category")["rating"].transform(
        lambda x: x.fillna(x.median())
    )
    global_median = df["rating"].median()
    df["rating"] = df["rating"].fillna(global_median)

    print(f"Missing values remaining: {df.isnull().sum().sum()}")
    return df


def standardize_brands(df: pd.DataFrame) -> pd.DataFrame:
    """Strip whitespace and title-case brand names for consistency."""
    df["brand"] = df["brand"].str.strip().str.title()
    return df


def fix_dtypes(df: pd.DataFrame) -> pd.DataFrame:
    """Ensure price columns are float64; handle any ₹ or comma symbols."""
    for col in ["sale_price", "market_price"]:
        if df[col].dtype == object:
            df[col] = (
                df[col]
                .astype(str)
                .str.replace("₹", "", regex=False)
                .str.replace(",", "", regex=False)
            )
            df[col] = pd.to_numeric(df[col], errors="coerce")
        df[col] = df[col].astype("float64")
    return df


def flag_anomalies_and_dedup(df: pd.DataFrame) -> pd.DataFrame:
    """Flag rows where sale_price > market_price; remove full duplicates."""
    df["is_price_anomaly"] = df["sale_price"] > df["market_price"]
    print(f"Price anomalies flagged: {df['is_price_anomaly'].sum()}")

    before = len(df)
    df = df.drop_duplicates().reset_index(drop=True)
    print(f"Duplicates removed: {before - len(df)}")
    return df


# ---------------------------------------------------------------------------
# Feature engineering
# ---------------------------------------------------------------------------

def engineer_features(df: pd.DataFrame) -> pd.DataFrame:
    """Create derived columns used in EDA, statistical analysis, and Tableau."""
    # Discount metrics
    df["discount_amount"] = (df["market_price"] - df["sale_price"]).round(2)
    df["discount_pct"] = np.where(
        df["market_price"] > 0,
        ((df["market_price"] - df["sale_price"]) / df["market_price"] * 100).round(2),
        0.0,
    )

    # Price segment
    df["price_segment"] = pd.cut(
        df["sale_price"],
        bins=[-np.inf, 100, 500, np.inf],
        labels=["Budget", "Mid-range", "Premium"],
    )

    # Is discounted flag
    df["is_discounted"] = df["sale_price"] < df["market_price"]

    # Rating segment
    df["rating_segment"] = pd.cut(
        df["rating"],
        bins=[-np.inf, 3, 4, np.inf],
        labels=["Low", "Medium", "High"],
    )

    print(
        f"Feature engineering complete. "
        f"New columns: discount_amount, discount_pct, price_segment, "
        f"is_discounted, rating_segment"
    )
    return df


# ---------------------------------------------------------------------------
# Validation
# ---------------------------------------------------------------------------

def validate(df: pd.DataFrame) -> None:
    """Print a short post-cleaning validation report."""
    print("\n=== Post-Cleaning Validation ===")
    print(f"Shape       : {df.shape[0]:,} rows × {df.shape[1]} columns")
    print(f"Missing     : {df.isnull().sum().sum()}")
    print(f"Duplicates  : {df.duplicated().sum()}")
    print(f"Columns     : {df.columns.tolist()}")


# ---------------------------------------------------------------------------
# I/O helpers
# ---------------------------------------------------------------------------

def build_clean_dataset(input_path: Path) -> pd.DataFrame:
    """Full pipeline: load raw CSV → clean → feature-engineer → return df."""
    print(f"Loading raw data from: {input_path}")
    df = pd.read_csv(input_path)
    print(f"Raw shape: {df.shape}")

    df = normalize_columns(df)
    df = drop_non_analytical(df)
    df = handle_missing_values(df)
    df = standardize_brands(df)
    df = fix_dtypes(df)
    df = flag_anomalies_and_dedup(df)
    df = engineer_features(df)

    validate(df)
    return df


def save_processed(df: pd.DataFrame, output_path: Path) -> None:
    """Write cleaned DataFrame to disk, creating parent directories if needed."""
    output_path.parent.mkdir(parents=True, exist_ok=True)
    df.to_csv(output_path, index=False)
    print(f"\nSaved cleaned dataset to: {output_path}")
    print(f"Rows: {len(df):,} | Columns: {len(df.columns)}")


# ---------------------------------------------------------------------------
# CLI entry-point
# ---------------------------------------------------------------------------

def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(
        description="Run the BigBasket ETL pipeline (Capstone 2)."
    )
    parser.add_argument(
        "--input",
        required=True,
        type=Path,
        help="Path to the raw CSV file (e.g. data/raw/BigBasket_Products.csv).",
    )
    parser.add_argument(
        "--output",
        required=True,
        type=Path,
        help="Destination for the cleaned CSV (e.g. data/processed/bigbasket_cleaned.csv).",
    )
    return parser.parse_args()


def main() -> None:
    args = parse_args()
    cleaned_df = build_clean_dataset(args.input)
    save_processed(cleaned_df, args.output)


if __name__ == "__main__":
    main()