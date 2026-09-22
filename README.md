# NHL Win-Factor Analysis

An end-to-end data engineering and analytics capstone built in Microsoft Fabric to identify which in-game performance factors are most strongly associated with winning an NHL game.

The project transforms raw game, team, skater and goalie data into a validated star schema, five analytical Delta tables and a four-page Power BI report. The results describe associations with winning; they do not establish causation.

## Business questions

- Which performance factors are most strongly associated with winning?
- How do winning teams differ from losing teams?
- How does win rate change across low-to-high performance bands?
- Do winner-loser differences vary between the regular season and playoffs?
- Have the main winner-loser performance gaps changed across seasons?

## Data source and scope

Source: [NHL Game Data by Martin Ellis on Kaggle] (https://www.kaggle.com/datasets/martinellis/nhl-game-data)

The source data spans the 2000/01 to 2019/20 period, covering 19 played seasons. All 13 source CSV files were retained in the Bronze layer. Six Silver tables were used for the analysis:

Silver table | Grain | Rows |
'silver_game' | One row per game | 23,735 |
'silver_game_teams_stats' | One row per game per team | 47,462 |
'silver_team_info' | One row per team | 33 |
'silver_game_skater_stats' | One row per game per skater | 853,404 |
'silver_player_info' | One row per player | 3,925 |
'silver_game_goalie_stats' | One row per game per goalie | 51,163 |

The raw dataset is not included in this repository. Download it from Kaggle and follow its stated usage terms.

## Technology stack

- Microsoft Fabric Lakehouse and OneLake
- Dataflow Gen2 and Power Query
- Fabric Notebook with PySpark
- Delta Lake tables
- Power BI semantic model and DAX
- Medallion architecture: Bronze, Silver and Gold

## Data pipeline

A[Kaggle CSV files] --> B[Bronze: raw files in OneLake]
B --> C[Silver: Dataflow Gen2 and Power Query]
C --> D[Gold: PySpark and Delta tables]
D --> E[Power BI semantic model]
E --> F[Four-page analytical report]

### Bronze layer

- Retained all 13 source CSV files in their original form.
- Used OneLake as the central storage location.

### Silver layer

- Corrected and promoted column headers.
- Assigned appropriate text, integer, decimal, date and datetime types.
- Handled confirmed duplicates using the relevant business keys.
- Converted genuine unavailable historical statistics to null rather than zero.
- Removed unused linkage columns that did not support the analysis.
- Profiled the resulting tables for valid, error and empty-value percentages.

### Gold layer

- Restricted the main analysis to regular-season and playoff games.
- Aggregated skater and goalie records to one row per game per team before joining them.
- Engineered scoring, shooting, special-teams, physical-play, puck-management, player-contribution and goaltending measures.
- Created analysis-scope flags to keep the full history while supporting fair comparisons over periods with complete metrics.
- Stored the curated outputs as Delta tables.

## Data-quality decisions

The source tables operate at different grains and contain several historical data limitations. The following rules were applied:

Issue | Treatment |

- Fourteen conditional playoff fixtures were never played | Excluded from the Gold analytical population |
- Five All-Star exhibition games used nonstandard team IDs and formats | Excluded from competitive-game analysis |
- Historical games could end in ties | Retained in Gold but excluded from binary winner-versus-loser comparisons |
- Seven completed team-game records had missing goalie data | Retained with goalie-derived fields left null |
- Faceoff percentage was unavailable through 2009/10 | Restricted complete-factor comparisons to 2010/11 onwards |
- Hits, giveaways, takeaways and blocked shots were unavailable in 2000/01 and 2001/02 | Preserved as null and documented |
- Player and goalie tables contained multiple records per team-game | Aggregated before joining to prevent row multiplication |

### Final analytical population

- 23,716 completed competitive games
- 47,432 team-game records
- One row per game per team
- 628 historical tied games retained in Gold
- 25,292 team-game records in the complete-factor analysis window

## Gold data model

The semantic model uses a central team-game fact table connected to four dimensions:

- 'gold_fact_team_game'
- 'gold_dim_team'
- 'gold_dim_opponent'
- 'gold_dim_date'
- 'gold_dim_season'

The fact table contains one row per game per team. Team, opponent, date and season dimensions filter the fact table through one-to-many relationships.

The wide curated table, 'gold_nhl_win_factors', was retained in the Lakehouse for transformation and validation but hidden from report users in the semantic model.

## Analytical outputs

Five derived Delta tables answer specific dashboard questions:

Table | Purpose |
- 'gold_analysis_01_winner_loser_averages' | Compare average metrics for winning and losing teams |
- 'gold_analysis_02_win_factor_correlations' | Rank Pearson correlations with the binary win flag |
- 'gold_analysis_03_performance_bands' | Compare win rates across four performance bands |
- 'gold_analysis_04_regular_vs_playoffs' | Compare winner-loser gaps by game stage |
- 'gold_analysis_05_season_trends' | Track winner-loser gaps across seasons |

Complete-factor comparisons use the 2010/11 to 2019/20 analysis window. Pearson correlation with a binary outcome is interpreted as a descriptive association, not evidence of causation.

## Power BI report

The four report pages are:

1. **Win-Factor Overview** — ranked correlations and winner-versus-loser averages.
2. **Performance Bands** — win rates across quartiles of the selected factor.
3. **Regular Season vs Playoffs** — winner and loser averages by game stage.
4. **Season Trends** — changes in the selected winner-loser gap by season.

Each analytical page displays the 2010/11 to 2019/20 comparison window. Metric selectors allow users to explore different performance factors.

## Key findings

- Shooting percentage had the strongest measured association with winning among the factors analysed, with a correlation of **0.58**.
- Goal-scorer count (**0.57**), point-contributor count (**0.55**) and team save percentage (**0.54**) also showed positive associations.
- Home-team status had a comparatively small correlation with winning (**0.09**).
- For shooting percentage, win rate increased from **11.26%** in the lowest performance band to **88.74%** in the highest band.
- The shooting-percentage winner-loser gap was **6.70 percentage points** during the regular season and **6.93 percentage points** in the playoffs.
- Hits, penalty minutes and giveaways showed weak or near-zero individual relationships with winning.

These findings are descriptive. Shooting percentage includes goals, which directly determine game outcomes, so it should not be interpreted as an independent causal driver.

## Validation

The project included the following checks:

- Uniqueness at each intended table grain
- Null-key and referential-integrity checks
- Two team rows per completed game
- Reconciliation of winning, losing and historical tie outcomes
- Verification that joins did not multiply or remove team-game records
- Missing-data profiling by season
- Range checks for percentages and nonnegative performance statistics
- Foreign-key validation between the fact and dimension tables
- Row-count validation for all five analytical outputs
- Power BI slicer, cross-filter, season-ordering and reset-state testing

## Limitations

- The analysis measures association and does not establish causation.
- Shooting percentage contains goals, which directly determine outcomes.
- Missing historical metrics restrict complete comparisons to 2010/11 onwards.
- Playoff samples are smaller and include only teams that qualified.
- Pearson correlation captures linear association only.
- Performance bands contain approximately equal team-game counts; tied or rounded values may appear in adjacent bands.

## Potential extensions

- Automate ingestion and add run-level data-quality logging.
- Add opponent-strength and score-state context.
- Extend the analysis to play-by-play data.
- Evaluate logistic-regression and tree-based models.
- Add player and game-stage dimensions if the reporting scope expands.

## Author

Keeris Wu
