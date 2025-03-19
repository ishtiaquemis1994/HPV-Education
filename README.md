# Analysis of HPV Learning Effectiveness and Resource Feedback

This document describes two Python scripts (likely part of Jupyter Notebooks) that analyze participant performance and feedback related to HPV (Human Papillomavirus) education. The first script processes pre- and post-learning test scores from `deidentified-pre-learning.xlsx` and `deidentified-post-learning.xlsx` to assess learning improvement. The second script analyzes participant feedback on learning resources from `quality.xlsx` to evaluate their perceived quality.

---

## Part 1: Pre- and Post-Learning Score Analysis

### 1.1 Setup and Dependencies
- **Libraries Used**:
  - `pandas`: For data manipulation and analysis.
  - `matplotlib.pyplot`: For basic plotting capabilities.
  - `seaborn`: For enhanced statistical visualizations (specifically a boxplot).
  - `google.colab.drive`: To mount Google Drive for accessing files in Google Colab.
- **Initial Step**: The script mounts Google Drive to access the data files.

### 1.2 File Paths and Data Loading
- **Files**:
  - `deidentified-pre-learning.xlsx`: Contains responses from the pre-learning test (baseline).
  - `deidentified-post-learning.xlsx`: Contains responses from the post-learning test.
- **Action**: These Excel files are loaded into pandas DataFrames:
  - `df_pre`: Pre-learning data.
  - `df_post`: Post-learning data.

### 1.3 Correct Answers Definition
- **Purpose**: A dictionary (`correct_answers`) stores correct answers for 19 multiple-choice questions about HPV.
- **Examples**:
  - "What does HPV stand for?" → "b) Human Papillomavirus"
  - "How is HPV primarily transmitted?" → "c) Through sexual contact"
- **Role**: Used as the reference for scoring participant responses.

### 1.4 Data Pre-processing
- **Cleaning**: Column names in both DataFrames are stripped of extra spaces using `.str.strip()`.
- **Filtering**: Only columns matching the test questions (keys in `correct_answers`) are extracted:
  - `df_pre_filtered`: Pre-learning responses.
  - `df_post_filtered`: Post-learning responses.

### 1.5 Score Calculation
- **Function**: `calculate_scores` compares responses to `correct_answers` and counts correct answers per participant.
- **Execution**:
  - Pre-learning scores are added as a "Score" column to `df_pre`.
  - Post-learning scores are added as a "Score" column to `df_post`.
- **Averages**:
  - `avg_pre_score`: Mean pre-learning score.
  - `avg_post_score`: Mean post-learning score.

### 1.6 Visualization
- **Data Preparation**: Scores are combined into `df_combined` with a "Category" column ("Pre-learning" or "Post-learning").
- **Plot**:
  - A boxplot (`seaborn.boxplot`) compares score distributions:
    - **X-axis**: "Pre-learning" vs. "Post-learning".
    - **Y-axis**: Scores.
    - **Title**: "Comparison of Participant Scores Before and After Learning".
    - **Styling**: Colors (`#3498db` for pre, `#e74c3c` for post), grid lines, font adjustments.
  - Displayed with `plt.show()`.

### 1.7 Purpose
- **Objective**: Assess the effectiveness of a learning intervention by comparing HPV knowledge before and after.
- **Outcome**: Quantify and visualize learning improvement.

### 1.8 Output
- **Results**:
  - Individual participant scores for both phases.
  - Average scores for pre- and post-learning.
  - A boxplot showing score distributions.

### 1.9 Context
- **Data**: Deidentified test responses from participants.
- **Methodology**: Participants took the same 19-question test twice—before and after a learning process.

---

## Part 2: Learning Resource Feedback Analysis

### 2.1 Setup and Dependencies
- **Imported Libraries**:
  - `pandas`: For data manipulation and DataFrame operations.
  - `scipy.stats`: For independent t-tests (`stats.ttest_ind`).
- **Assumption**: Runs in an environment like Google Colab, given the file path.

### 2.2 Data Loading
- **File**: 
  - `quality.xlsx`: Located at `/content/drive/MyDrive/HPV/quality.xlsx`.
- **Action**: Loaded into `df_quality` using `pd.read_excel`.

### 2.3 Data Pre-processing
- **Likert Columns**: 
  - Columns from index 2 onward contain Likert-scale responses, skipping non-numeric columns.
  - Stored in `likert_columns`.
- **Conversion**: Converted to numeric with `pd.to_numeric`, `errors='coerce'` turning non-numeric entries into `NaN`.

### 2.4 Descriptive Statistics Calculation
- **Overall Statistics**:
  - Computed for `likert_columns` using `.describe().T`.
  - "Positive Impression (%)" calculated as percentage of responses ≥ 4: `(df_quality[likert_columns] >= 4).mean() * 100`.
  - "Category" column added as "Overall".
  - Stored in `grouped_stats["Overall"]`.
- **Grouped Statistics**:
  - Grouped by `"Which resources you have been assigned by your professor"`.
  - For each resource ("Booklet", "Internet search", "Chat GPT", "Visual storytelling"):
    - If present, Likert responses are extracted and converted to numeric.
    - Descriptive stats and positive impression percentages are computed.
    - "Category" column added with resource name.
    - Stored in `grouped_stats[resource]`.
- **Merging**: 
  - Combined into `df_grouped_stats` using `pd.concat`, with columns renamed: `"level_1"` → `"Question"`, `"level_0"` → `"Category"`.

### 2.5 Statistical Analysis (T-Tests)
- **Objective**: Compare positive impression rates (≥ 4) per resource against overall data.
- **Process**:
  - `p_values_results` initialized with `likert_columns` as "Question".
  - For each resource:
    - Group data extracted and converted to numeric.
    - For each column:
      - Binary indicators created: 1 for ≥ 4, 0 otherwise (`overall_positive`, `resource_positive`).
      - Welch’s t-test (`stats.ttest_ind`, `equal_var=False`, `nan_policy='omit'`) performed.
      - `p_value` set to `None` if insufficient data.
    - P-values stored in `p_values_results[resource]`.
  - Missing resources assigned `None` for all p-values.
- **DataFrame Conversion**: 
  - P-values reshaped into `df_p_values` via `melt`: `"Question"`, `"Category"`, `"P-Value"`.

### 2.6 Final Table Construction
- **Type Consistency**: 
  - `"Question"` and `"Category"` columns cast to strings in both DataFrames.
- **Merging**: 
  - `df_grouped_stats` and `df_p_values` merged on `"Question"` and `"Category"` into `df_final`.
- **Output**: 
  - `df_final` displayed, including stats, positive impression percentages, and p-values.

### 2.7 Purpose
- **Objective**: Evaluate perceived quality of learning resources for HPV education.
- **Analysis**: Quantify feedback and test for significant differences from overall impressions.

### 2.8 Output
- **Result**: `df_final` with:
  - Descriptive stats per question and category.
  - Positive impression percentages.
  - P-values for resource vs. overall comparisons.

### 2.9 Context
- **Data**: Likert-scale feedback on resources in `quality.xlsx`.
- **Methodology**: Assesses satisfaction with assigned learning tools.

---

## Combined Summary
These scripts collectively analyze HPV education:
- **Score Analysis**: Uses `deidentified-pre-learning.xlsx` and `deidentified-post-learning.xlsx` to measure knowledge improvement via scores and a boxplot.
- **Feedback Analysis**: Uses `quality.xlsx` to evaluate resource quality with stats and t-tests.
Together, they provide a data-driven assessment of learning effectiveness and resource perception among participants.
