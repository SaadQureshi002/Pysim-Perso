# SIM Provisioning Suite

A Python-based **GSM / USIM / SIM personalization data generation system** with both a desktop GUI and a command-line workflow.

The project generates structured SIM personalization batches containing subscriber identifiers, authentication material, administrative credentials, and OTA keysets. It also provides automatic ICCID/IMSI sequencing, issuance protection, batch history, verification utilities, and formatted Microsoft Excel exports.

> **Security notice:** This project can generate cryptographic SIM key material. Treat configuration files and generated outputs as sensitive data. Do not commit production `settings.json` files or real operator keys to public repositories.

---

## Features

- Desktop **Tkinter GUI** for SIM batch generation
- Command-line batch generation with `run_batch.py`
- Automatic **ICCID and IMSI range calculation**
- Automatic suggestion of the next unused ICCID/IMSI range
- Unique batch IDs in the format `BATCH-YYYYMMDD-####`
- Issuance ledger protection against accidental identifier reuse
- Configurable batch size and output file name
- Configurable PIN1, PIN2, PUK1, and PUK2 values
- SIM authentication and OTA key generation
- OPc, EKI, and ACC derivation
- Microsoft Excel `.xlsx` export
- Professionally formatted `SIM_DATA` and `BATCH_INFO` worksheets
- Batch history tracking
- End-to-end verification script
- Standards-oriented GSM/USIM encoding and personalization workflow

---

## What the System Generates

A generated SIM record may contain the following data:

| Category | Fields |
|---|---|
| Subscriber identifiers | `ICCID`, `IMSI` |
| Authentication | `KI`, `OPC`, `EKI`, `ACC` |
| Cardholder credentials | `PIN1`, `PIN2`, `PUK1`, `PUK2` |
| Administrative data | `ADM1`, `ADM6` |
| OTA keysets | `KIC1-3`, `KID1-3`, `KIK1-3` |
| Operator parameters | `OP`, `K4` |

The core generator can produce `ELECT`, `SERVER`, and `GRAPH` datasets depending on the configuration.

---

## Standards Referenced

The underlying GSM data-generation library follows or references:

| Area | Standard |
|---|---|
| OPc derivation / MILENAGE | 3GPP TS 35.206 |
| IMSI, ICCID and ACC EF encoding | 3GPP TS 31.102 |
| Access control classes | 3GPP TS 22.011 |
| ICCID numbering / Luhn check digit | ITU-T E.118 |
| OTA keysets | ETSI TS 102 225 |

The verification utility includes a deterministic OPc test vector based on **3GPP TS 35.206**.

---

## Requirements

- **Python 3.10 or newer**
- Windows is recommended for the current GUI because it uses `os.startfile()` to open generated files and folders.
- `pip`

Major runtime dependencies include:

- `pandas`
- `openpyxl`
- `pydantic`
- `pycryptodome`
- `numpy`
- `python-dateutil`
- `pytz`

The package dependencies are installed through the project setup configuration.

---

## Installation

Clone or download the project and open a terminal in the repository root.

### 1. Create a virtual environment

```bash
python -m venv .venv
```

Activate it on Windows:

```bash
.venv\Scripts\activate
```

On Linux/macOS:

```bash
source .venv/bin/activate
```

### 2. Install the project

For a normal installation:

```bash
pip install .
```

For development:

```bash
pip install -e .
```

### 3. Create the local configuration

Copy the example configuration:

**Windows**

```bash
copy settings.example.json settings.json
```

**Linux/macOS**

```bash
cp settings.example.json settings.json
```

Then edit `settings.json` with the required operator and batch parameters.

> `settings.json` is intentionally excluded from version control because it may contain sensitive operator and cryptographic values.

---

## Configuration

The configuration file contains three main sections.

### `DISP`

Controls generation parameters such as:

```json
{
  "DISP": {
    "imsi": "<starting IMSI>",
    "iccid": "<starting ICCID>",
    "pin1": "1234",
    "puk1": "12345678",
    "pin2": "5678",
    "puk2": "87654321",
    "size": 10,
    "prod_check": true,
    "elect_check": true,
    "graph_check": true,
    "server_check": false
  }
}
```

The real configuration also contains operator cryptographic values such as `op` and `K4`. Do not publish real production values.

### `PATHS`

Controls output naming and location:

```json
{
  "PATHS": {
    "FILE_NAME": "my_batch",
    "OUTPUT_FILES_DIR": "output",
    "OUTPUT_FILES_LASER_EXT": "laser"
  }
}
```

### `PARAMETERS`

Controls which fields are written to each output and how laser-marking fields are sliced.

Examples include:

- `data_variables`
- `server_variables`
- `laser_variables`

---

# Running the Application

## Option 1 — Desktop GUI

Start the desktop application with:

```bash
python gui_app.py
```

The window opens as:

**SIM Provisioning Suite**

The GUI allows the operator to enter or review:

- Starting ICCID
- Starting IMSI
- Number of SIMs
- Output file name
- PIN1
- PUK1
- PIN2
- PUK2

The application automatically previews the ending ICCID and IMSI based on the selected batch size.

### GUI Batch Workflow

When a batch is generated, the application:

1. Loads and validates `settings.json`.
2. Determines the next available ICCID and IMSI.
3. Validates the entered batch parameters.
4. Generates a unique batch ID.
5. Creates the SIM personalization records.
6. Writes the generated batch through the core issuance-protected pipeline.
7. Creates a Microsoft Excel workbook.
8. Formats the workbook for operator use.
9. Removes the intermediate TXT files created during the GUI workflow.
10. Saves the batch in local batch history.
11. Loads the next unused ICCID/IMSI range for the following batch.

Generated GUI filenames follow the pattern:

```text
<file-name>_BATCH-YYYYMMDD-####.xlsx
```

Example:

```text
PAF_LTE_0012_BATCH-20260907-0001.xlsx
```

---

## Excel Output

The GUI produces a formatted Excel workbook.

### `SIM_DATA`

The main generated dataset is stored in the `SIM_DATA` worksheet.

The GUI applies formatting including:

- Styled column headers
- Frozen header row
- Auto-filtering
- Alternating row shading
- Automatic column widths
- Text formatting for ICCID, IMSI and cryptographic fields
- Preserved leading digits/zeros where applicable

### `BATCH_INFO`

A second worksheet named `BATCH_INFO` records operational metadata such as:

- Batch ID
- Generation date and time
- Number of SIMs
- Starting ICCID
- Ending ICCID
- Starting IMSI
- Ending IMSI
- Export format
- Issuance protection status

This provides a simple audit/reference sheet alongside the generated personalization data.

---

## Automatic Identifier Sequencing

The system automatically calculates the ending range using:

```text
end = start + quantity - 1
```

After a successful issuance, the next batch can start from:

```text
Previous Ending ICCID + 1
Previous Ending IMSI  + 1
```

This allows consecutive batches to be produced without manually calculating the next identifier range.

---

## Issuance Protection

Issued ranges are tracked through:

```text
.issuance_ledger.json
```

inside the configured output directory.

The ledger is used to prevent accidental reuse or overlap of previously issued ICCID ranges.

This is important because regenerating the same identifier range may create the same ICCIDs with different cryptographic keys, resulting in invalid or conflicting personalization data.

Do not manually delete the issuance ledger in a production workflow unless you fully understand the consequences.

---

## Batch History

The GUI maintains a local history file:

```text
.batch_history.json
```

The history is used for batch tracking and for generating sequential daily batch IDs.

Batch IDs follow:

```text
BATCH-YYYYMMDD-####
```

For example:

```text
BATCH-20260907-0001
BATCH-20260907-0002
BATCH-20260907-0003
```

The sequence restarts for a new date.

---

# Command-Line Batch Generation

A command-line workflow is also available:

```bash
python run_batch.py
```

The script loads `settings.json` and prompts for:

```text
Starting ICCID
Starting IMSI
Number of SIMs
Output file name
PIN1
PUK1
PIN2
PUK2
```

Press **Enter** at a prompt to keep the displayed default value.

Unlike the GUI workflow, `run_batch.py` keeps the normal generated TXT outputs and additionally creates an Excel workbook.

Example workflow:

```text
============================================================
        GSM / SIM DATA GENERATOR - EXCEL EXPORT
============================================================

Enter batch details.
Press ENTER to use the value shown in brackets.

Starting ICCID [...]
Starting IMSI [...]
Number of SIMs [10]
Output file name [my_batch]
PIN1 [...]
PUK1 [...]
PIN2 [...]
PUK2 [...]
```

After generation, the script displays the generated frames, output TXT paths, and Excel file location.

---

# Verification

Run:

```bash
python verify.py
```

The verification utility performs four groups of checks:

1. **Package imports**
2. **Random data generators**
3. **Cryptographic and encoding operations**
4. **Full generation pipeline**

Among other checks, it validates:

- Ki generation
- OTA key generation
- PIN generation
- PUK generation
- OPc calculation
- EKI calculation
- ACC calculation
- XOR behavior
- PIN encode/decode round trip
- 3GPP TS 35.206 OPc test vector
- Configuration parameter validation
- Generated DataFrames

Use a custom configuration:

```bash
python verify.py --config path/to/settings.json
```

Skip the complete pipeline test:

```bash
python verify.py --no-pipeline
```

A successful run exits with code `0`; one or more failed checks return code `1`.

---

# Output Modes

The underlying generator supports three output types.

| Output | Intended Use |
|---|---|
| `ELECT` | SIM personalization / electrical personalization equipment |
| `SERVER` | HLR/HSS or provisioning-side data |
| `GRAPH` | Laser marking / printed SIM information |

These outputs can be individually enabled or disabled in `settings.json`.

The desktop GUI converts generated data into its formatted Excel workflow and removes the intermediate TXT files after successful Excel creation. The CLI batch script retains the standard TXT outputs and adds an Excel export.

---

# Project Structure

A typical repository layout is:

```text
.
├── gsm_data_generator/        # Core GSM/SIM generation library
├── gui_app.py                 # Desktop SIM Provisioning Suite
├── run_batch.py               # Interactive CLI + Excel export
├── verify.py                  # End-to-end verification utility
├── settings.example.json      # Safe configuration template
├── settings.json              # Local configuration (do not commit)
├── gen_requirements.py        # Dependency/requirements generator
├── setup.py                   # Package configuration
├── pyproject.toml             # Python build-system configuration
├── MANIFEST.in                # Source distribution manifest
├── mypy.ini                   # Static type-checking configuration
├── LICENSE                    # Apache License 2.0
└── README.md
```

Generated output directories may also contain:

```text
.issuance_ledger.json
.batch_history.json
*.xlsx
*.txt
```

depending on the workflow being used.

---

# Core Generation Flow

At application level, the main generation flow is:

```python
from gsm_data_generator import DataGenerationScript, json_loader

config = json_loader("settings.json")

script = DataGenerationScript(config)
script.json_to_global_params()

result_dfs, keys = script.generate_all_data()

written = script.write_outputs(result_dfs)
```

The GUI and CLI build their respective workflows around this core API.

---

# Security Considerations

This project handles security-sensitive SIM personalization information.

Follow these practices:

1. Never commit a populated `settings.json`.
2. Never expose real `KI`, `K4`, `OP`, `OPC`, OTA keys, PINs, or PUKs in screenshots, issues, logs, or public repositories.
3. Keep generated Excel/TXT files on protected storage.
4. Restrict access to issuance ledgers and generated personalization data.
5. Use test values for demonstrations and development.
6. Do not reuse issued ICCID/IMSI ranges.
7. Treat generated files as confidential even when used only for testing.

The repository `.gitignore` excludes `settings.json` and generated output paths to reduce the risk of accidental commits.

---

# Packaging

The project uses `setuptools`.

Build-system configuration is defined in `pyproject.toml`, with `setuptools` as the backend.

To build distributable packages:

```bash
python -m pip install build
python -m build
```

The package is published/configured under the Python package name:

```text
gsm-data-generator
```

and requires Python `>=3.10`.

---

# Development

Install in editable mode:

```bash
pip install -e .
```

Useful development checks include:

```bash
python verify.py
```

If the repository includes its test suite, additional checks may be run with tools such as `pytest`, `black`, and `mypy`.

---

# Troubleshooting

### `ModuleNotFoundError: No module named 'gsm_data_generator'`

Install the project from the repository root:

```bash
pip install -e .
```

### `settings.json` not found

Create it from the example configuration:

```bash
copy settings.example.json settings.json
```

or on Linux/macOS:

```bash
cp settings.example.json settings.json
```

### Batch overlaps an already issued range

Use the next available ICCID/IMSI suggested by the application. Check the issuance ledger before manually changing ranges.

### Excel file will not open from the GUI

Confirm the generated `.xlsx` file still exists in the configured output directory.

### GUI does not open

Confirm that:

- Python 3.10+ is installed
- Tkinter is available
- project dependencies are installed
- `settings.json` is valid
- the command is being run from the repository root

---

# License

Licensed under the **Apache License 2.0**.

See [`LICENSE`](LICENSE) for details.

---

## Disclaimer

This software is intended for development, testing, research, lab, and authorized SIM-personalization workflows. Operators are responsible for protecting subscriber identifiers, authentication secrets, and generated provisioning material and for complying with applicable telecommunications, privacy, security, and organizational requirements.
