## Day 1 - [date]

**Goal:** Set up the project and explore the data

**What I did**
- Created the GitHub repo, cloned it, and set up the virtual environment in VS Code
- Downloaded the Kaggle credit card fraud dataset and loaded it in `01_eda.ipynb`
- Made 4 EDA charts (class balance, amount by class, fraud rate by hour, top features)

**Key findings**
- 284,807 rows, 31 columns, 0 missing values
- 492 frauds (0.1727%), about 578 normal transactions per fraud
- Data is already sorted by Time
- Amount: [what the boxplot showed]
- Time of day: [what the hourly chart showed]

**Decisions and why**
- Will split by time, not randomly, because the data is time-ordered and a random split would leak future patterns
- Will judge models by PR-AUC and money saved, not accuracy

**Problems and fixes**
- `touch` and multi-folder `mkdir` failed in PowerShell: used `New-Item` and comma-separated folder names instead

**Next**
- Day 2: time-based split, metrics, and baselines