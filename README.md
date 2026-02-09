# NFL Stadium Attendance Regression Analysis

A multi-language data analysis project examining predictors of NFL home game attendance using mixed-effects regression modeling. This project demonstrates proficiency in SQL, Python, and R by performing data integration, cleaning, and statistical analysis across all three languages.

## Project Context

This project was originally developed as a final project for **ST541 (Statistics for Business Analytics)** at the University of Alabama's Culverhouse College of Business. While the course focused on R and regression modeling, our team extended the project to demonstrate competency across the full data science toolkit: SQL for data integration, Python for data cleaning and feature engineering, and R for statistical modeling.

## Research Question

**What factors significantly predict NFL home game attendance?**

We investigate team performance metrics, temporal patterns, winning/losing streaks, and team fixed effects while controlling for year and week random effects to identify the strongest predictors of stadium attendance.

## Key Technical Achievements

- **Multi-Language Integration**: Seamlessly combines SQL (data joins), Python (cleaning/feature engineering), and R (statistical modeling)
- **Temporal Precedence Handling**: Implements lagged variables (1-week and 1-year lags) to avoid using post-game statistics to predict pre-game attendance
- **Custom Feature Engineering**: Calculates consecutive win/loss streaks by iterating through chronological game data
- **Mixed-Effects Modeling**: Compares team fixed effects with year/week fixed and random effects to find optimal model structure
- **Multicollinearity Treatment**: Identifies and removes highly correlated predictors before backward elimination

## Statistical Findings

Our analysis revealed that the optimal model includes:
- **Team fixed effects**: Controls for team-specific factors (market size, fan loyalty, stadium capacity)
- **Year random effects**: Accounts for temporal variation across seasons
- **Lowest AIC/BIC scores**: Indicates best model fit while penalizing complexity

After removing multicollinear variables and applying backward elimination, we identified significant predictors of attendance while maintaining model parsimony.

## Quick Start

This project uses [Pixi](https://pixi.sh/) for environment management and [Marimo](https://marimo.io/) for interactive Python notebooks.

### Installing Pixi

Pixi is a modern, cross-platform package manager that handles Python dependencies and virtual environments automatically.

**macOS/Linux:**
```bash
curl -fsSL https://pixi.sh/install.sh | bash
```

**Windows (PowerShell):**
```powershell
iwr -useb https://pixi.sh/install.ps1 | iex
```

**Alternative (using Homebrew on macOS):**
```bash
brew install pixi
```

After installation, restart your terminal or run:
```bash
source ~/.bashrc  # Linux
source ~/.zshrc   # macOS
```

### Running the Analysis

1. **Clone this repository** and navigate to the project directory

2. **Install dependencies**:
   ```bash
   pixi install
   ```

3. **Run the interactive notebook**:
   ```bash
   pixi run marimo edit nfl-regression-analysis.py
   ```
   
   This will launch an interactive Marimo notebook in your web browser at `http://localhost:2718`

### Data Files

All required data files are **included in this repository**:
- `Attendance.csv` - Weekly attendance figures for NFL home games
- `games.csv` - Game-level statistics (scores, yards, turnovers, etc.)
- `standings.csv` - Season-end team statistics and rankings

**No additional downloads required!** The analysis runs immediately after `pixi install`.

## Project Structure

```
.
├── nfl-data-cleaning.py    # Main Marimo notebook (SQL + Python)
├── Attendance.csv                # Weekly attendance data
├── games.csv                     # Game-level statistics
├── standings.csv                 # Season standings and team metrics
├── final_dataframe/              # Generated output directory
│   └── attendance_with_lags.csv  # Clean dataset for R analysis
├── pixi.toml                     # Pixi dependency configuration
├── pixi.lock                     # Locked dependency versions
└── README.md                     # This file
```

## Data Pipeline

### Part 1: Data Integration (SQL)

Uses Marimo's native SQL support to perform multi-table joins:

1. **First join**: Combines `Attendance.csv` with `standings.csv` on team name and year
2. **Second join**: Adds `games.csv` data matching team, year, and week
3. **Result**: Unified dataset with attendance, game outcomes, and season statistics

**SQL Implementation:**
- Common Table Expressions (CTEs) for readable query structure
- Left joins to preserve all attendance records
- Chronological ordering by team, year, and week

### Part 2: Feature Engineering (Python)

Implements sophisticated data cleaning and variable creation using Polars:

#### Consecutive Win/Loss Streaks
- **Method**: Chronological iteration through all games (home and away)
- **Logic**: Tracks each team's current win/loss streak at game time
- **Challenge**: Requires processing full game history before filtering to home games only
- **Implementation**: Uses dictionaries to maintain team-specific streak counters

#### Lagged Variables (Temporal Precedence)
- **1-week lags**: Previous game statistics (points, yards, turnovers, etc.)
- **1-year lags**: Prior season metrics (wins, playoff appearance, rankings)
- **Rationale**: Attendance is decided before kickoff, so we can't use same-week game outcomes as predictors

#### Dummy Variable Creation
- **Day of week**: Converts categorical days to numeric (Sun=0, Mon=1, ..., Sat=6)
- **Tie indicator**: Binary flag for tie games
- **Home win**: Binary outcome (1=home team won, 0=away team won)

#### Data Type Conversions
- Ensures numeric columns are properly typed for statistical analysis
- Handles missing values appropriately
- Maintains data integrity through transformation pipeline

### Part 3: Statistical Modeling (R)

The cleaned dataset (`attendance_with_lags.csv`) is exported for regression analysis in R:

1. **Model Comparison**:
   - Team fixed effects vs. random effects
   - Year fixed effects vs. random effects
   - Week fixed effects vs. random effects
   - Model selection via AIC/BIC minimization

2. **Multicollinearity Diagnostics**:
   - Variance Inflation Factor (VIF) calculation
   - Removal of highly correlated predictors
   - Ensures stable coefficient estimates

3. **Backward Elimination**:
   - Iteratively removes non-significant predictors
   - Optimizes model based on AIC/BIC
   - Final model includes only significant, non-collinear predictors

## Analysis Workflow

### 1. Load Data and Initial Joins

```python
# Uses Marimo SQL support
full_data_df = mo.sql("""
    WITH attendance_standings AS (
        SELECT a.*, s.* 
        FROM attendance_df a 
        LEFT JOIN standings_df s 
            ON a.full_name = s.full_name 
            AND a.year = s.year
    )
    SELECT * FROM attendance_standings
    LEFT JOIN games_df g 
        ON attendance_standings.team_name = g.home_team_name
        ...
""")
```

### 2. Generate Streak Variables

```python
# Python iteration to calculate consecutive wins/losses
team_win_streaks = {}
for row in games_df.iter_rows(named=True):
    if home_won:
        team_win_streaks[home_team] += 1
        team_loss_streaks[home_team] = 0
    # ... (streak logic)
```

### 3. Create Lagged Features

```python
# Polars window functions for lag operations
clean_df = df.with_columns([
    pl.col('weekly_attendance').shift(1).over(['home_team', 'year']).alias('weekly_attendance_lag'),
    pl.col('pts_win').shift(1).over(['home_team', 'year']).alias('pts_win_lag'),
    # ... (additional lags)
])
```

### 4. Export for R Analysis

```python
# Write final cleaned dataset
final_clean_full_dataframe.write_csv('final_dataframe/attendance_with_lags.csv')
```

The final dataframe is generated through the data cleaning process in the Python notebook and exported as a CSV file ready for statistical modeling in R.

## Dataset Overview

**Source**: [NFL Stadium Attendance Dataset](https://www.kaggle.com/datasets/sujaykapadnis/nfl-stadium-attendance-dataset) (Kaggle)  
**Coverage**: Multiple NFL seasons with weekly game-level data  
**Granularity**: One observation per home game  

### Key Variables

**Dependent Variable:**
- `weekly_attendance` - Number of fans at home game

**Independent Variables (Lagged):**
- **Game Performance**: Points scored/allowed, yards gained/allowed, turnovers
- **Season Metrics**: Wins, losses, point differential, strength of schedule
- **Rankings**: Offensive ranking, defensive ranking, Simple Rating System (SRS)
- **Streaks**: Consecutive wins, consecutive losses
- **Temporal**: Day of week, week number, year
- **Team Status**: Prior year playoff appearance

## Technologies

### Core Stack

- **Python 3.14**: Latest Python release
- **Marimo**: Reactive notebooks with native SQL support
- **Polars**: High-performance DataFrame library
- **DuckDB**: Embedded SQL engine (via Marimo)
- **R**: Statistical modeling and regression analysis
- **Pixi**: Cross-platform package manager

### Why This Stack?

**Marimo for Multi-Language Analysis:**
- Native SQL support (no separate database required)
- Seamless Python integration
- Reactive execution ensures consistency
- Pure Python files (`.py` not `.ipynb`) for version control

**Polars for Data Cleaning:**
- Lazy evaluation for memory efficiency
- Expressive window functions for lag operations
- Fast iteration capabilities for streak calculations
- Modern API with method chaining

**R for Statistical Modeling:**
- Industry standard for mixed-effects models
- Comprehensive regression diagnostics
- `lme4` package for random effects
- AIC/BIC model comparison tools

## Variable Descriptions

### Temporal Precedence Logic

A critical consideration in this analysis: **attendance is determined before kickoff**, so we cannot use same-game statistics as predictors. All game-level variables are lagged by 1 week, and all season-level variables are lagged by 1 year.

**Example:**
- Week 5 attendance is predicted using Week 4 game statistics
- 2023 attendance is predicted using 2022 season metrics

### Variable Categories

**Lagged Game Statistics (1 week):**
- `pts_win_lag`, `pts_loss_lag` - Points scored by winner/loser last week
- `yds_win_lag`, `yds_loss_lag` - Yards gained by winner/loser last week
- `turnovers_win_lag`, `turnovers_loss_lag` - Turnovers committed last week
- `win_streak_lag`, `lose_streak_lag` - Consecutive W/L before this game
- `day_dummy_lag` - Day of week from last game

**Lagged Season Statistics (1 year):**
- `lag_wins`, `lag_loss` - Prior season record
- `lag_points_for`, `lag_points_against` - Prior season scoring
- `lag_simple_rating` - Prior year SRS rating
- `lag_offensive_ranking`, `lag_defensive_ranking` - Prior year ranks
- `lag_playoffs_dummy` - Made playoffs previous year (1/0)

**Current Variables (Known Pre-Game):**
- `home_team`, `away_team` - Team identifiers
- `year`, `week` - Temporal identifiers
- `weekly_attendance` - **Dependent variable**

## Development Notes

### Understanding the Data Flow

1. **Raw CSVs** → Loaded as Polars DataFrames
2. **SQL Joins** → Combined via Marimo's SQL engine
3. **Python Processing** → Feature engineering and lagging
4. **CSV Export** → Clean data for R analysis
5. **R Modeling** → Regression analysis and diagnostics

### Extending the Analysis

To add new variables or modify the pipeline:

1. **New predictors**: Add to the SQL SELECT statements or create in Polars
2. **Different lags**: Modify `.shift()` parameters in the lagging section
3. **Additional joins**: Extend the CTE structure in the SQL query
4. **New models**: Import the CSV into R and specify new model formulas

### Marimo Cell Structure

The notebook uses `@app.cell` decorators for reactive execution:

```python
@app.cell
def _(dependency1, dependency2):
    result = process(dependency1, dependency2)
    return result,  # Note: comma makes it a tuple
```

Cells automatically re-run when dependencies change, ensuring data consistency.

## Key Insights from Course Project

This analysis demonstrated several important concepts from ST541:

1. **Fixed vs. Random Effects**: Team effects are fixed (32 teams are the full population), while year/week effects are random (sample from infinite possible years/weeks)

2. **Multicollinearity**: Variables like `points_for` and `offensive_ranking` are highly correlated, requiring careful variable selection

3. **Model Selection**: AIC/BIC provide objective criteria for comparing non-nested models

4. **Temporal Structure**: Panel data (multiple observations per team over time) requires careful modeling of correlation structure

5. **Practical Significance**: Statistical significance doesn't always equal practical importance - small effects may be significant with large sample sizes

## Reproducing the R Analysis

After running the Python notebook to generate `attendance_with_lags.csv`:

```r
# Load cleaned data
df <- read.csv("final_dataframe/attendance_with_lags.csv")

# Example: Team fixed effects + Year random effects
library(lme4)
model <- lmer(weekly_attendance ~ 
              win_streak_lag + 
              lag_wins + 
              lag_simple_rating + 
              home_team +  # Team fixed effects
              (1|year),    # Year random effects
              data = df)

# Model diagnostics
summary(model)
AIC(model)
BIC(model)
```

## Future Enhancements

Potential extensions to this analysis:

- **Weather data**: Temperature, precipitation effects on attendance
- **Rivalry games**: Indicator for divisional/historic matchups
- **Ticket pricing**: Economic factors affecting attendance
- **COVID-19 analysis**: 2020 pandemic effects on attendance patterns
- **Forecasting models**: Time series predictions for future attendance
- **Geographic analysis**: Distance between team cities, regional effects

## Course Context

**Course**: ST541 - Statistics for Business Analytics  
**Institution**: University of Alabama Culverhouse College of Business  
**Semester**: Fall 2025  
**Team Approach**: Collaborative project demonstrating SQL, Python, and R proficiency  

**Learning Objectives Demonstrated**:
- Mixed-effects regression modeling
- Panel data analysis techniques
- Multi-language data pipeline development
- Feature engineering and temporal precedence
- Model comparison and selection
- Handling multicollinearity

## License

This project is available for educational purposes. Dataset is provided by Kaggle user Sujay Kapadnis and subject to Kaggle's terms of use.

## Data Source Attribution

**Original Dataset**: [NFL Stadium Attendance Dataset](https://www.kaggle.com/datasets/sujaykapadnis/nfl-stadium-attendance-dataset)  
**Provider**: Sujay Kapadnis (Kaggle)  
**License**: Subject to Kaggle's licensing terms

## Acknowledgments

- **Dataset**: Sujay Kapadnis for compiling and sharing NFL data on Kaggle
- **Course**: Dr. Yaofang Hu for answering all my nonsense questions throughout this project :)
- **Tools**: Marimo, Polars, and R development communities

---

**Questions or Issues?**

If you encounter any problems running this analysis or have questions about the methodology, please open an issue on GitHub.
# nfl-regression-analysis
