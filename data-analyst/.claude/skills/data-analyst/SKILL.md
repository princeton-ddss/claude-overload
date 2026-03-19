---
name: data-analyst
description: >
  Use when the user asks a question that can be answered with data.
  Performs ad-hoc analysis: explores data, selects methods, produces results.
  Examples: "What predicts turnout?", "How do these groups differ?",
  "Is there a trend in poverty rates?"
---

# Data Analyst

Answer research questions using appropriate statistical methods.

## Workflow

1. **Understand the question**
   - What is being asked? (comparison, prediction, relationship, description)
   - What are the key variables? (outcome, predictors, grouping)
   - What is the unit of analysis?

2. **Check data availability**
   - Check `data/sources.yaml` and `data/clean/` to see what's available
   - If data is missing, invoke the `data-manager` skill to ingest it
   - If datasets need linking, invoke the `match` skill
   - If spatial operations are needed, invoke the `geo` skill

3. **Analyze and deliver**

   Default mode unless the user asks for a notebook or interactive exploration.

   ### Default mode
   Run the analysis and report results directly.

   - Write analysis scripts to `scripts/` (or `.tmp/` for throwaway work)
   - Run the scripts and report findings in conversation
   - Include key numbers, tables, and interpretation
   - Tell the user where to find the script(s)

   ### Interactive mode
   Use when the user asks for a notebook, wants to explore interactively,
   or says "open in marimo" / "make it interactive".

   - Create the notebook following the `marimo-notebook` skill conventions
   - Save to `notebooks/`
   - Open for the user using `run_in_background: true`: `marimo edit --watch <file>`

## Parallel experiments

For robustness checks, model comparisons, or exploring multiple hypotheses,
spawn multiple `experimenter` agents in parallel. Each runs one specification
and returns structured results. Synthesize the findings yourself.

Examples:
- Test the same model on different subsets (by region, time period, demographic)
- Compare model specifications (add/remove controls, different functional forms)
- Run sensitivity analyses (drop outliers, alternative variable definitions)
- Test multiple hypotheses from a single research question

## Question types

| Type | Example | Methods |
|------|---------|---------|
| Describe | "What's the distribution of income?" | Summary stats, histograms |
| Compare | "How does turnout differ by region?" | t-test, ANOVA, group means |
| Relate | "Is education correlated with income?" | Correlation, scatterplot |
| Predict | "What factors predict turnout?" | Regression (linear, logistic) |
| Trend | "How has poverty changed over time?" | Time series, slope |
| Cluster | "Are there natural groupings?" | k-means, PCA |
