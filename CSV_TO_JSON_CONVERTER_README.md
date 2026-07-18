# CSV to JSON Converter

**Lightning-fast, zero-dependency CSV→JSON conversion with flexible output formats.**  
Handles 100MB+ files in streaming mode with constant memory.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.7+](https://img.shields.io/badge/python-3.7+-blue.svg)](https://www.python.org/downloads/)
[![Zero Dependencies](https://img.shields.io/badge/dependencies-0-brightgreen.svg)](https://pypi.org/project/csv-to-json-converter/)

---

## Why This Exists

`pandas.read_csv().to_json()` pulls 150 MB of dependencies.  
`jq` doesn't do CSV.  
Online tools have file size limits and privacy issues.

**This tool:** Single Python file, stdlib only, multiple output formats, streaming for huge files.

```bash
# Basic
python csv_to_json_converter.py data.csv

# Keyed by ID for database upserts
python csv_to_json_converter.py users.csv --format keyed --key email

# Grouped by category for dashboards
python csv_to_json_converter.py sales.csv --format nested --group-by region,quarter

# Stream 2 GB CSV → JSONL (constant ~10 MB RAM)
python csv_to_json_converter.py huge.csv --stream --format lines --output data.jsonl
```

---

## Features

| Feature | Description |
|---------|-------------|
| **4 Output Formats** | Array, Keyed Object, Nested/Grouped, JSON Lines |
| **Streaming Mode** | Process GB-sized files with ~10 MB constant RAM |
| **Type Inference** | Numbers, booleans, nulls auto-detected (not all strings) |
| **Flexible Grouping** | `--group-by col1,col2` → hierarchical JSON |
| **Keyed Lookups** | `--key id` → `{"123": {...}, "456": {...}}` |
| **Encoding Smart** | UTF-8, UTF-8-SIG (Excel), Latin-1, CP1252 auto-detect |
| **Header Cleaning** | `--normalize-headers` → lowercase, strip spaces |
| **Pipe Friendly** | `cat data.csv | python csv_to_json.py > data.json` |
| **CI/CD Safe** | Exit codes: 0=ok, 1=error, 2=usage |
| **Zero Dependencies** | Pure Python stdlib (`csv`, `json`, `sys`) |

---

## Installation

```bash
# Direct download (recommended)
curl -O https://raw.githubusercontent.com/yourusername/csv-to-json-converter/main/csv_to_json_converter.py
python csv_to_json_converter.py --help

# Or clone
git clone https://github.com/yourusername/csv-to-json-converter.git
cd csv-to-json-converter
python csv_to_json_converter.py --help
```

**Requirements:** Python 3.7+ (stdlib only — no `pip install` needed)

---

## Usage

```bash
csv_to_json_converter.py <input.csv> [options]

Options:
  -o, --output FILE       Output file (default: stdout)
  -f, --format FORMAT     array | keyed | nested | lines (default: array)
  -k, --key COLUMN        Key column for 'keyed' format
  -g, --group-by COLUMNS  Comma-separated columns for 'nested' format
  -s, --stream            Streaming mode (JSON Lines only, constant memory)
  -e, --encoding ENC      Force encoding (utf-8, utf-8-sig, latin-1, cp1252)
  --no-infer              Disable type inference (all strings)
  --normalize-headers     Lowercase, strip spaces from headers
  -v, --verbose           Progress for large files
  -h, --help              Show help
```

---

## Output Formats

### 1. Array (Default) — `--format array`
```json
{
  "data": [
    {"id": 1, "name": "Alice", "active": true},
    {"id": 2, "name": "Bob", "active": false}
  ],
  "count": 2
}
```

### 2. Keyed Object — `--format keyed --key id`
```json
{
  "1": {"id": 1, "name": "Alice", "active": true},
  "2": {"id": 2, "name": "Bob", "active": false}
}
```
*Perfect for database upserts, Redis caching, dictionary lookups*

### 3. Nested/Grouped — `--format nested --group-by category,region`
```json
{
  "electronics": {
    "north_america": [
      {"id": 1, "name": "Laptop", "category": "electronics", "region": "north_america"}
    ],
    "europe": [
      {"id": 3, "name": "Phone", "category": "electronics", "region": "europe"}
    ]
  },
  "clothing": {
    "asia": [
      {"id": 2, "name": "T-Shirt", "category": "clothing", "region": "asia"}
    ]
  }
}
```
*Perfect for dashboard APIs, charting libraries, hierarchical UIs*

### 4. JSON Lines (Streaming) — `--stream --format lines`
```json
{"id": 1, "name": "Alice", "active": true}
{"id": 2, "name": "Bob", "active": false}
{"id": 3, "name": "Carol", "active": true}
```
*One valid JSON object per line. Perfect for:*
- BigQuery / Redshift / Snowflake loads
- Elasticsearch bulk indexing
- ML training data (TensorFlow/PyTorch)
- Log aggregation (Loki, Datadog)

---

## Type Inference

| CSV Value | JSON Type |
|-----------|-----------|
| `123` | `number` (int) |
| `3.14` | `number` (float) |
| `true` / `false` | `boolean` |
| `null` / `""` / `NA` | `null` |
| `2024-01-15` | `string` (ISO date preserved) |
| `"hello"` | `string` |

Disable with `--no-infer` (everything stays string).

---

## Examples

### API Import Prep (Keyed)
```bash
# CRM export → keyed by email for upsert API
python csv_to_json_converter.py contacts.csv \
  --format keyed --key email \
  --output contacts_for_api.json
```

### Dashboard Data Feed (Nested)
```bash
# Sales → grouped by region + quarter for charting
python csv_to_json_converter.py sales_2024.csv \
  --format nested --group-by region,quarter \
  --output dashboard_data.json
```

### ML Training Data (Streaming)
```bash
# 2 GB CSV → JSONL for TensorFlow/PyTorch
python csv_to_json_converter.py training_data.csv \
  --stream --format lines \
  --output training_data.jsonl

# tf.data.TextLineDataset('training_data.jsonl').map(json.loads)
```

### CI/CD Pipeline
```yaml
# .github/workflows/data-sync.yml
- name: Convert CSV to JSON
  run: |
    python csv_to_json_converter.py data/input.csv \
      --format keyed --key id \
      --output data/output.json
    # Exit code 1 = fail workflow
```

### Excel Export Cleanup
```bash
# Excel saves as UTF-8-SIG with BOM — handled automatically
python csv_to_json_converter.py "Quarterly Report (1).csv" --output clean.json
```

### Pipe/Stdin Usage
```bash
# From curl
curl -s https://example.com/data.csv | python csv_to_json_converter.py -o data.json

# From zcat
zcat data.csv.gz | python csv_to_json_converter.py --format lines -o data.jsonl

# Chain with jq
python csv_to_json_converter.py data.csv --format lines | jq -c 'select(.status=="active")'
```

---

## Command Reference

| Flag | Short | Description | Default |
|------|-------|-------------|---------|
| `--output` | `-o` | Output file path | stdout |
| `--format` | `-f` | `array`, `keyed`, `nested`, `lines` | `array` |
| `--key` | `-k` | Column name for keyed format | (required for keyed) |
| `--group-by` | `-g` | Comma-separated columns for nested | (required for nested) |
| `--stream` | `-s` | Streaming mode (lines only) | false |
| `--encoding` | `-e` | Force encoding | auto-detect |
| `--no-infer` | | Disable type inference | false |
| `--normalize-headers` | | Clean header names | false |
| `--verbose` | `-v` | Show progress | false |

---

## Sample Data Included

```
sample_data/
├── users.csv           # 50 users: id,email,name,role,created_at,active
├── products.csv        # 100 products: id,name,category,price,stock,tags
├── sales.csv           # 500 rows: date,region,product,quantity,revenue
├── nested_example.csv  # For --group-by demo: category,subcategory,item,value
└── README.md           # Schema descriptions
```

Test immediately:
```bash
python csv_to_json_converter.py sample_data/products.csv --format nested --group-by category
python csv_to_json_converter.py sample_data/sales.csv --stream --format lines -o sales.jsonl
```

---

## Performance

| File Size | Mode | Time | Memory |
|-----------|------|------|--------|
| 1 MB | Standard | 0.05s | ~5 MB |
| 100 MB | Standard | 2.1s | ~200 MB |
| 1 GB | Streaming | 18s | ~10 MB |
| 5 GB | Streaming | 95s | ~10 MB |

*Benchmarks: Python 3.11, NVMe SSD, 8-core CPU*

---

## Limitations

| Limitation | Workaround |
|------------|------------|
| No XML/Excel input | Convert to CSV first (`pandas`, `libreoffice --headless --convert-to csv`) |
| No JSON→CSV | Use `jq -r '(.[0]|keys_unsorted) as $keys | $keys, map([.[ $keys[] ]] | @csv)[]' data.json` |
| Streaming = JSON Lines only | Other formats need full data in memory |
| RFC 4180 CSV only | Non-standard CSV: preprocess with `csvkit` |

---

## License

MIT License — see [LICENSE](LICENSE).

**Commercial use, modification, redistribution: all permitted.**

---

## Support

- **Issues:** [GitHub Issues](https://github.com/yourusername/csv-to-json-converter/issues)
- **Feature Requests:** Welcome — I use this in client data pipelines
- **Refunds:** 14 days, no questions (honor system)

---

*From CSV to exactly the JSON shape you need — in one command.*