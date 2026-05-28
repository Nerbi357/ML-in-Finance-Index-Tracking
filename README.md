# ML-in-Finance-S-P500-Tracking
# Sparse Index Tracking on the S&P 500

Can you replicate the S&P 500 with just a handful of stocks instead of all 500? This project builds and compares three ways of doing exactly that — picking a small basket of stocks that moves like the index — and then asks a harder question: which method actually holds up once you account for the cost of trading?

## The Idea

Owning the whole S&P 500 means holding 500 positions and constantly rebalancing them. That is expensive and impractical for a small portfolio. **Sparse index tracking** is the art of choosing a much smaller set of stocks — say 10, 30, or 50 — and weighting them so the basket rises and falls almost in step with the full index.

This is a classic trade-off: fewer stocks are cheaper and simpler to manage, but harder to keep aligned with the index. The project explores where that balance lands.

## Three Methods

The project implements and compares three approaches, from simplest to most sophisticated:

**Market-cap Top-K (baseline).** Just take the K largest companies in the index and weight them by size. No optimization, no cleverness — the benchmark any smarter method should beat.

**LASSO with cardinality control.** A machine-learning approach that looks at how stocks have moved historically and selects a small group whose combined behavior best matches the index. It uses L1 regularization to automatically zero out unhelpful stocks, with a search procedure to hit the target number of holdings.

**SLAIT (Benidis et al., 2018).** A specialized algorithm that solves the tracking problem directly, enforcing the exact number of holdings as a hard constraint. It uses the original authors' R package, called from Python, so the results match the published method exactly.

All three share the same testing framework, so the comparison is fair.

## How It Was Tested

The methods are evaluated with a **walk-forward backtest**: at every rebalance the model is retrained on the most recent year of data and held for the following month, rolling forward through a decade of history. The data is split into an in-sample period (for developing and calibrating the methodology) and an out-of-sample period (held back entirely for a final, unbiased test). The framework avoids common backtesting traps — it only uses information available at each point in time and includes companies that later left the index, so the results are not flattered by hindsight.

## Key Findings

- **Tracking generalizes.** Performance on the held-out period closely matched the development period, which means the methods genuinely learned to track rather than overfitting to past data.

- **SLAIT tracks best at small portfolios.** When holding very few stocks, the specialized algorithm replicated the index noticeably more tightly than the other two. LASSO struggled badly with very small baskets but caught up once allowed more stocks.

- **Transaction costs change everything.** The two optimization-based methods rebalance aggressively — they reshuffle their holdings far more than the simple baseline. Once realistic trading costs are applied, their apparent advantage shrinks or disappears, while the stable, low-turnover baseline barely flinches. The lesson: a method that looks great on paper can be expensive to actually run.

- **Beating the index is not the goal.** Over the test period several portfolios outperformed the S&P 500, but that is a sign of *deviation*, not good tracking. The point of an index tracker is to match the benchmark, not to bet against it — and by that standard, closeness matters more than returns.

## Repository Structure

```
notebooks/
  01_data_collection.ipynb   Data gathering, cleaning, and exploratory analysis
  02_modeling.ipynb          The three methods, framework, and in-sample backtest
  03_evaluation.ipynb        Out-of-sample test, cost sensitivity, final figures
reports/
  (final written report)
```

Each notebook is self-contained and documents every methodological choice as it goes. They are meant to be read in order, but each can be run on its own.

## Data and Tools

Built with Python (pandas, NumPy, CVXPY) and R (the `sparseIndexTracking` package via rpy2), with price data from Yahoo Finance and point-in-time index membership from a public historical-constituents dataset. The benchmark is the S&P 500 Total Return index.

## Notes and Limitations

This is an academic project. A few simplifications are documented in the notebooks — for example, current share counts are used as a proxy for historical market caps, and weights are allowed to drift between rebalances (a standard convention). These are noted openly rather than hidden, in keeping with the project's emphasis on honest, reproducible methodology.

##AI Tool Disclosure. This project was developed with the assistance of 
Anthropic's Claude (claude.ai), used for the following purposes:
- Code drafting and debugging
- Methodology discussion
- Markdown narrative and English-language editing

All methodological decisions, parameter choices, and interpretations of 
results were made by the author. Code was reviewed and tested before 
inclusion. The final analysis, methodology, and conclusions are the 
author's responsibility.
