# BESS Optimizer — Nord Pool SE3

Battery Energy Storage System dispatch optimizer using real Swedish 
electricity market data, ML price forecasting, and Linear Programming.

## Results
- **94.9% average revenue capture rate** over 76-day backtest
- **10.89% MAPE** on LightGBM price forecast
- Tested on real Nord Pool SE3 day-ahead prices (Sweden)

## What it does
1. Pulls real-time day-ahead prices from Nord Pool SE3
2. Forecasts next 24h prices using LightGBM
3. Optimizes BESS charge/discharge schedule using Linear Programming
4. Backtests performance over historical data

## Tech Stack
Python | LightGBM | Pyomo | HiGHS | Plotly | Pandas

## Author
Racem Kamel — Renewable Energy Engineer  
Exchange semester at Mälardalen University (MDU), Västerås, Sweden — 2026  
Target thesis: Hitachi Energy grid automation division
