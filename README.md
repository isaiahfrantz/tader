# lets build a stock bot

https://vectorbt.pro

https://github.com/polakowo/vectorbt.pro/blob/pvt-links/README.md

## chat
asked
> help create a sample backtesting trading bot with this repo in a python notebook. it must pull 1 year of 1 minute bars for s&p500 from alpaca, using my api key, and save them locally for future runs. it must watch trigger on macd and slow stochastics crosses 

answer:
To create a sample backtesting trading bot using the `vectorbt.pro` repository, we can follow these steps in a Python notebook:

1. **Set up the environment:**
   - Install the necessary dependencies (`vectorbt`, `alpaca-trade-api`, and others).
   - Connect to Alpaca using your API key to fetch 1-minute bars for the S&P 500 over the past year.

2. **Download and save data:**
   - Save the historical data locally as a CSV file for future runs.

3. **Define the indicators:**
   - Compute the MACD (Moving Average Convergence Divergence) and Slow Stochastics indicators using `vectorbt`.

4. **Set up the backtesting logic:**
   - Define the trading strategy based on MACD and Slow Stochastics crossovers.

5. **Run the backtest:**
   - Use `vectorbt` to simulate the strategy and analyze the results.

Here's the notebook code to achieve this:

```python
# Install required libraries
# !pip install vectorbt alpaca-trade-api pandas

import os
import pandas as pd
import vectorbt as vbt
from alpaca_trade_api.rest import REST, TimeFrame

# Alpaca API credentials (replace with your own API key and secret)
ALPACA_API_KEY = "your_api_key"
ALPACA_API_SECRET = "your_api_secret"
ALPACA_BASE_URL = "https://paper-api.alpaca.markets"

# Initialize Alpaca API
alpaca = REST(ALPACA_API_KEY, ALPACA_API_SECRET, ALPACA_BASE_URL)

# Fetch 1 year of 1-minute bars for S&P 500
symbol = "SPY"  # ETF representing S&P 500
data_file = f"{symbol}_1min.csv"

if not os.path.exists(data_file):
    print("Fetching historical data from Alpaca...")
    bars = alpaca.get_bars(
        symbol,
        TimeFrame.Minute,
        start=pd.Timestamp.now() - pd.Timedelta(days=365),
        end=pd.Timestamp.now()
    ).df
    bars = bars[bars['symbol'] == symbol]  # Filter for the requested symbol
    bars.to_csv(data_file)
else:
    print("Loading historical data from local file...")
    bars = pd.read_csv(data_file, index_col='timestamp', parse_dates=True)

# Convert to vectorbt DataFrame
price = vbt.Data.load_pandas(bars['close'])

# Define indicators
macd = vbt.MA.run(price, window=12).ma - vbt.MA.run(price, window=26).ma  # MACD line
signal = vbt.MA.run(macd, window=9).ma  # Signal line
macd_crossover = macd > signal  # MACD crossover

stoch_k = (price - price.rolling(14).min()) / (price.rolling(14).max() - price.rolling(14).min()) * 100
stoch_d = stoch_k.rolling(3).mean()
stoch_crossover = stoch_k > stoch_d  # Stochastic crossover

# Define strategy: Buy when both MACD and Stochastics cross upwards
entries = macd_crossover & stoch_crossover
exits = ~entries

# Backtest
portfolio = vbt.Portfolio.from_signals(price, entries, exits)
portfolio.performance()
portfolio.plot().show()
```

### Key Features of This Notebook:
1. **Alpaca Integration:** Automatically fetches and saves historical data locally to avoid repeated API calls.
2. **MACD and Slow Stochastics Indicators:** Implements both indicators to define buy and sell signals.
3. **Backtesting with `vectorbt`:** Simulates the strategy and displays performance metrics.

Let me know if you need further adjustments!
