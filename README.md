# PRGI Publication Title Similarity Checker

A Python-based tool that screens a proposed publication title against a
dataset of existing publication titles. It checks exact duplicates,
potentially prohibited combinations of existing titles, spelling
similarity, and (when available) semantic similarity.

## Features

-   **Input validation:** rejects empty, malformed, or very short
    titles.
-   **Generic-word screening:** flags titles made up of a single
    configured generic root word.
-   **Exact duplicate detection:** compares normalized titles against
    the dataset.
-   **Title-combination detection:** identifies when a proposed title
    contains two or more existing registered titles as consecutive word
    sequences.
-   **TF-IDF candidate retrieval:** uses character-level n-grams and
    cosine similarity to retrieve likely matches.
-   **Jaro--Winkler comparison:** checks spelling similarity and can
    flag likely typos or minor spelling variations.
-   **Optional SBERT comparison:** uses the `all-MiniLM-L6-v2`
    Sentence-BERT model for semantic similarity when
    `sentence-transformers` is installed.
-   **Domain keyword conflict check:** identifies configured contrastive
    word pairs and sends those cases for review.
-   **Explainable recommendations:** prints a recommendation and the
    reason for it.

## Project Files

Keep these files in the same project directory:

``` text
publication-title-checker/
├── duplicate_check.py
├── publictaion_titles_real.xls
└── README.md
```

> The dataset filename is intentionally written as
> `publictaion_titles_real.xls` here because that is the filename
> currently used in the Python script. If you rename the dataset, update
> the filename in `duplicate_check.py` as well.

## Requirements

-   Python 3.9 or later
-   pandas
-   NumPy
-   scikit-learn
-   RapidFuzz
-   lxml (used by pandas to read the HTML table stored in the dataset
    file)
-   Optional: sentence-transformers and its PyTorch dependencies for
    SBERT comparison

## Installation

1.  Open a terminal in the project directory.
2.  Create and activate a virtual environment (recommended).

**Windows**

``` bash
python -m venv venv
venv\Scripts\activate
```

**macOS / Linux**

``` bash
python3 -m venv venv
source venv/bin/activate
```

3.  Install the required packages:

``` bash
pip install pandas numpy scikit-learn rapidfuzz lxml
```

4.  To enable semantic comparison with SBERT, install the optional
    package:

``` bash
pip install sentence-transformers
```

The first run with SBERT may download the `all-MiniLM-L6-v2` model and
therefore requires an internet connection. If `sentence-transformers` is
not installed, the program continues without SBERT and uses the
Jaro--Winkler score for its final similarity decision.

## Dataset

The script reads `publictaion_titles_real.xls` with `pandas.read_html()`
and takes the second column (column index `1`) as the publication title.
Although the file has an `.xls` extension, its contents are an HTML
table, not a conventional binary Excel workbook.

The dataset must therefore: - remain in the project directory, or the
path in the script must be updated; - contain an HTML table readable by
`pandas.read_html()`; and - have publication titles in the second
column.

## How to Run

Run the program from the project directory:

``` bash
python duplicate_check.py
```

The current script evaluates the sample title configured at the bottom
of the file:

``` python
evaluate_user_title("hindus times")
```

To test another title, replace the sample string with your proposed
title:

``` python
evaluate_user_title("Your Proposed Publication Title")
```

Save the file and run the command again.

## How the Screening Process Works

1.  **Load and prepare the dataset:** reads the publication-title
    column, removes missing/blank values, and removes duplicate titles.
2.  **Normalize titles:** converts text to lowercase, replaces
    punctuation with spaces, and collapses repeated whitespace.
3.  **Validate the input:** rejects empty titles and titles shorter than
    three characters after cleaning.
4.  **Check generic root words:** rejects a one-word title if it appears
    in the configured `GENERIC_ROOT_WORDS` set.
5.  **Check exact duplicates:** rejects a title if its normalized form
    already exists in the dataset.
6.  **Check title combinations:** flags a title for review if it
    contains at least two existing titles as exact consecutive word
    sequences.
7.  **Retrieve candidates with TF-IDF:** represents titles using
    character n-grams (`char_wb`, n-gram range 2--4), calculates cosine
    similarity, and keeps the top 20 candidates.
8.  **Compare spelling with Jaro--Winkler:** ranks the retrieved
    candidates by Jaro--Winkler similarity.
9.  **Check contrastive terms:** checks the configured pairs
    `detection/prevention`, `classification/prevention`, and
    `diagnosis/treatment`.
10. **Compare semantic similarity with SBERT when available:** unless
    the adaptive early-exit condition is met, compares the query with
    the top Jaro--Winkler candidate using `all-MiniLM-L6-v2`.
11. **Print a recommendation:** reports whether the title should be
    rejected, accepted, or flagged for manual review.

## Recommendation Categories

  --------------------------------------------------------------------------------------
  Recommendation                                     Meaning
  -------------------------------------------------- -----------------------------------
  `REJECT - INVALID INPUT`                           The cleaned input is empty or
                                                     shorter than three characters.

  `REJECT - DIRECT GUIDELINE VIOLATION`              The title consists only of one
                                                     configured generic root word.

  `REJECT - EXACT TITLE DUPLICATE`                   An identical normalized title is
                                                     already present.

  `FLAG FOR REVIEW - PROHIBITED TITLE COMBINATION`   The proposed title contains at
                                                     least two existing titles as word
                                                     sequences.

  `FLAG FOR REVIEW - HIGH SPELLING SIMILARITY`       Jaro--Winkler similarity reaches
                                                     the adaptive early-exit threshold
                                                     of 0.94, with no detected
                                                     contrastive-term conflict.

  `FLAG FOR REVIEW - DOMAIN KEYWORD CONFLICT`        A configured contrastive-term
                                                     conflict is detected.

  `FLAG FOR REVIEW - HIGH MULTI-SIGNAL SIMILARITY`   The final similarity score reaches
                                                     the configured threshold (default
                                                     0.85).

  `ACCEPT - NO SIMILARITY DETECTED`                  No retrieved candidate reaches the
                                                     minimum TF-IDF candidate score of
                                                     0.05.

  `ACCEPT - LIKELY UNIQUE TITLE`                     The final similarity score is below
                                                     the configured threshold.
  --------------------------------------------------------------------------------------

**Important:** `ACCEPT` is an automated screening recommendation, not an
official PRGI approval. A `FLAG FOR REVIEW` result should be checked by
a human reviewer.

## Main Configuration in the Script

You can adjust these values in `duplicate_check.py`:

-   `GENERIC_ROOT_WORDS`: the one-word generic terms screened by the
    program.
-   `CONTRASTIVE_PAIRS`: the configured pairs used by the domain
    conflict check.
-   `TfidfVectorizer(analyzer="char_wb", ngram_range=(2, 4))`: the
    character n-gram settings.
-   `EARLY_EXIT_THRESHOLD = 0.94`: the Jaro--Winkler threshold for the
    adaptive early exit.
-   `evaluate_user_title(user_title, threshold=0.85)`: the final
    similarity threshold.

## Technologies Used

-   Python
-   pandas and NumPy
-   scikit-learn: TF-IDF vectorization and cosine similarity
-   RapidFuzz: Jaro--Winkler similarity
-   Sentence-Transformers (optional): SBERT semantic embeddings

## Current Implementation Notes

-   The script currently prints results to the terminal; it does not
    provide a web interface or save a report file.
-   SBERT is optional. If it is unavailable, the semantic score remains
    `0.0` and the program continues with the Jaro--Winkler score.
-   The current implementation compares SBERT only with the single top
    Jaro--Winkler candidate, not every title in the dataset.
-   The final score is the maximum of the Jaro--Winkler and SBERT
    scores; it is not a trained probability or a calibrated confidence
    percentage.
-   The title-combination check looks for exact consecutive word
    sequences after normalization. It does not detect every possible
    paraphrase or rearrangement.
-   The configured generic words and contrastive pairs are project-level
    rules and should be reviewed against the applicable official
    guidelines before real-world use.

## Troubleshooting

**`FileNotFoundError` for the dataset** - Make sure
`publictaion_titles_real.xls` is in the same directory from which you
run the script. - Alternatively, update the path in `pd.read_html()`.

**`ImportError` for a Python package** - Activate the project virtual
environment and install the requirements listed above.

**SBERT model download or loading issue** - Check the internet
connection and try installing `sentence-transformers` again. - To run
without SBERT, leave `sentence-transformers` uninstalled.

**`ValueError: No tables found`** - Confirm that the dataset file is the
supplied HTML-table file and that it has not been replaced with a
different file format.


This project is an academic title-similarity screening tool. Its output
is intended to support preliminary review and should not be treated as a
legal decision, official registration decision, or a substitute for
checking the latest applicable PRGI rules and records.
