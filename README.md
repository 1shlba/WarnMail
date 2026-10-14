# WarnMail
Hybrid rule based/random forest integrated phishing-email detection tool. Small, lightweight, offline; targeted towards small business with limited resources and technical expertise.


Paste an email in, and WarnMail tells you whether it looks like phishing -
**Low**, **Medium** or **High** risk - and explains why in plain language. It
combines a hand-written **rule engine** with a **Random Forest** machine learning
model, so you get both specific, explainable warnings and a second opinion that
can spot combinations of signals.

> WarnMail is a **learning prototype**, built to demonstrate how a hybrid
> rules + machine-learning detector fits together. It is not a replacement for
> real email security, and it is deliberately trained on *synthetic* data. See
> [docs/LEARNING_GUIDE.md](docs/LEARNING_GUIDE.md) for the full picture.

---

## What it looks like

A suspicious email is rated **High risk**, with each warning sign spelled out:

```
========================================================================
WarnMail result: HIGH RISK  (score 96/100)
========================================================================
This looks like a phishing email. (8 rule-based warning signs; the model
rates it 94% likely phishing).

Warning signs found (8):
  [HIGH] A link is disguised
        The link text shows one website, but clicking it would take you
        somewhere else.
          - shows harbourlinebank.com.au but goes to 203.0.113.45
  [HIGH] Imitates a known organisation
  ...
What to do:
  This email has strong signs of phishing. Do not click any links...
```

A genuine invoice is rated **Low risk** with no warnings. Screenshots of the
app are in [docs/](docs/).

---

 ## Quick start (run from source)

You need **Python 3.12 or newer** (numpy and scipy require it).

```bash
# 1. Install the dependencies
python -m pip install -r requirements.txt
#    On Linux, also install Tkinter:  sudo apt install python3-tk

# 2. (first time only) create the dataset and train the model
python -m training.dataset
python -m training.train_model

# 3. Run the app
python app.py
```

On command line:

```bash
python -m warnmail.cli data/samples/01_phishing_bank_login.txt
python -m warnmail.cli --mode rules data/samples/03_phishing_bank_change_bec.txt
```

---

## Build a Windows .exe

On a Windows machine with Python 3.12 installed, just run:

```
build_exe.bat
```

It sets up a virtual environment, installs everything, trains a model if
needed, runs the tests and produces **`dist\WarnMail.exe`** — a single file you
can double-click, with no Python installation required to run it.

Behind the scenes that uses [`warnmail.spec`](warnmail.spec). To check a build
without a screen:

```
set WARNMAIL_SELFTEST=1 && dist\WarnMail.exe
```

which analyses the bundled samples and prints the results.

> **Why can't the .exe be built on Mac/Linux?** PyInstaller packages for the
> operating system it runs on. A Windows .exe must be built on Windows (or via
> a Windows CI runner — see the learning guide).

---

## How it works, in one picture

```
      pasted text / .eml file
               │
               ▼
        parse the email          (warnmail/parser.py)
               │
               ▼
      detect the signals         (warnmail/signals.py)
         │              │
         ▼              ▼
   turn into         apply the
    numbers           rules
 (features.py)       (rules.py)
         │              │
         ▼              ▼
   Random Forest    rule score + plain-language findings
   probability
         │              │
         └──────┬───────┘
                ▼
         blend + safety overrides     (warnmail/analyzer.py)
                │
                ▼
      Low / Medium / High + report     (warnmail/report.py, gui.py)
```

The three modes — **rules only**, **model only**, **hybrid** — can be compared
with `python -m training.evaluate` (see
[training/reports/evaluation.md](training/reports/evaluation.md)).

---

## Project layout

| Path | What it is |
| --- | --- |
| `app.py` | Launcher (opens the GUI). |
| `warnmail/` | The app: parser, signals, features, rules, model, analyzer, report, GUI, CLI. |
| `training/` | Dataset generator, model training, and the three-mode evaluation. |
| `data/` | The synthetic dataset, its dataset card, and sample emails. |
| `models/` | The trained Random Forest (`warnmail_rf.joblib`). |
| `tests/` | The test suite (`python -m pytest`). |
| `docs/` | Architecture notes and the learning guide. |
| `warnmail.spec`, `build_exe.bat` | Windows packaging. |
