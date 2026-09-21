# BIOSTAT 707 — Checkpoint 1

This project describes the PhysioNet Challenge 2012 set-a ICU cohort. 
The notebook includes data checks, Table 1, outcome summary, missingness plots, and a comparison of mortality by measurement availability.

## 1. Prepare the data

Install Pixi and place the supplied data in the project folder:

```text
data/
├── set-a/             # 4,000 ICU record files
│   ├── 132539.txt
│   └── ...
└── Outcomes-a.txt     # Outcome records
```

The notebook reads these files without downloading or modifying them.

## 2. Run the analysis

From the project folder, run:

```bash
pixi run --locked checkpoint1
```

This command prepares the environment, runs `checkpoint1.ipynb` from start to finish, and creates the report and supporting files in `output/`.

To open the notebook in JupyterLab:

```bash
pixi run lab
```

## 3. View the results

Open **`output/checkpoint1.html`** in a browser to read the complete report.

## 4. Before submission

Run this command once per clone to keep notebook outputs out of Git:

```bash
pixi run nbstripout --install --attributes .gitattributes
```

Commit `checkpoint1.ipynb`, `README.md`, `pixi.toml`, `pixi.lock`, `.gitignore`, `.gitattributes`, and the final `output/checkpoint1.html`. Do not commit raw data or other generated outputs
