# task-3
interactive dashboard
# ================================================================
# SNITCH FASHION SALES DATA CLEANING SCRIPT
# Author: [Your Name]
# Date: [Today's Date]
# ================================================================

import pandas as pd
from dateutil import parser

# 1️⃣ Load dataset
file_path = r"C:\Users\predator\Documents\nov internship\task 3\Snitch_Fashion_Sales_Uncleaned.csv"
df = pd.read_csv(file_path, encoding="utf-8")

print("✅ Original Data Loaded:", df.shape, "rows and columns")

# 2️⃣ Standardize column names
df.columns = df.columns.str.strip().str.title().str.replace(" ", "_")

# 3️⃣ Remove duplicate rows
df.drop_duplicates(inplace=True)

# 4️⃣ Clean text columns safely (no data loss)
for col in df.select_dtypes(include="object").columns:
    df[col] = df[col].astype(str).str.strip()
    df[col] = df[col].replace({"nan": None, "None": None, "": None})
    df[col] = df[col].apply(lambda x: x.title().strip() if isinstance(x, str) else x)

# 5️⃣ Handle multiple date formats safely
date_cols = [c for c in df.columns if "date" in c.lower()]
for col in date_cols:
    def parse_date_safe(x):
        try:
            return parser.parse(str(x), dayfirst=True)
        except Exception:
            return pd.NaT
    df[col] = df[col].apply(parse_date_safe)

# 6️⃣ Convert numeric-looking columns safely
for col in df.select_dtypes(include="object").columns:
    try:
        df[col] = df[col].str.replace(",", "")
        df[col] = pd.to_numeric(df[col], errors="raise")
    except Exception:
        continue

# 7️⃣ Fix city name variations and standardize spellings
if "City" in df.columns:
    df["City"] = df["City"].replace({
        "Hyd": "Hyderabad",
        "Hybd": "Hyderabad",
        "Hydrabad": "Hyderabad",
        "Hyd ": "Hyderabad",
        "Hyderbad": "Hyderabad",
        "Bangalore": "Bengaluru",
        "Banglore": "Bengaluru"
    })

# 8️⃣ Fill blank text fields with 'Unknown'
for col in df.select_dtypes(include="object").columns:
    df[col] = df[col].replace(["", " ", "nan", "NaN", "None"], pd.NA)
    df[col].fillna("Unknown", inplace=True)

# 9️⃣ Sort by Order ID if present
if "Order_Id" in df.columns:
    df.sort_values(by="Order_Id", inplace=True)

# 🔟 Save cleaned dataset
output_path = r"C:\Users\predator\Documents\nov internship\task 3\Snitch_Fashion_Sales_Cleaned_Final.csv"
df.to_csv(output_path, index=False, encoding="utf-8")

print("🎉 Cleaned dataset saved successfully at:", output_path)
print("✅ Final shape:", df.shape)

# 🧾 Optional: quick verification of cleaned city names
if "City" in df.columns:
    print("\nUnique cities after cleaning:\n", df["City"].unique())
