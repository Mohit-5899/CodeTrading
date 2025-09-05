# VWAP Breakout Trading Strategy

A comprehensive intraday trading strategy implementation using Volume Weighted Average Price (VWAP) as the primary signal for entry and exit decisions.

## Overview

This project implements a VWAP-based breakout strategy that:
- Takes long positions when price crosses above VWAP with bullish bias
- Takes short positions when price crosses below VWAP with bearish bias
- Includes ATR-based trailing stop loss management
- Automatically flattens positions before market close (15:45 ET)
- **NEW**: Supports Indian stock market data via yfinance integration

## Strategy Logic

### Entry Conditions
- **Long Entry**: Close price > VWAP AND Close price > Day's Open
- **Short Entry**: Close price < VWAP AND Close price < Day's Open

### Risk Management
- ATR-based trailing stop loss (configurable multiplier)
- Intraday position closure at 15:45 ET
- Single position at a time (exclusive orders)

### Key Features
- Daily and weekly VWAP calculation support
- **Indian stock market data support** via yfinance
- Parameter optimization capabilities
- Comprehensive backtesting with performance metrics
- Visual performance analysis
- Real-time data fetching from NSE/BSE

## Files Structure

- `vwap_stocks.ipynb` - Main notebook with Indian stock market integration
- `README.md` - This documentation file
- `requirements.txt` - Python dependencies

## Core Components

### 1. VWAP Calculation Functions

#### `add_vwap()`
Calculates daily VWAP (resets each calendar day):
- Uses typical price (High+Low+Close)/3 or specified price column
- Volume-weighted cumulative calculation
- Handles datetime parsing and indexing

#### `add_weekly_vwap()`
Calculates VWAP with custom frequency reset:
- Supports daily ("D"), weekly ("W-MON", "W-FRI"), monthly ("M") periods
- Flexible price column selection
- Configurable reset boundaries

### 2. Data Fetching Functions

#### `fetch_indian_stock_data()`
Fetches live Indian stock market data using yfinance:
- Supports NSE (.NS) and BSE (.BO) symbols
- Configurable time periods and intervals
- Automatic filtering of non-trading periods
- Returns clean OHLCV data with datetime index

**Example Usage:**
```python
# Fetch Reliance data from NSE
df = fetch_indian_stock_data("RELIANCE.NS", period="1y", interval="15m")

# Other popular Indian stocks
df = fetch_indian_stock_data("TCS.NS", period="6mo", interval="5m")
df = fetch_indian_stock_data("HDFCBANK.NS", period="2y", interval="1h")
df = fetch_indian_stock_data("INFY.NS", period="1y", interval="1d")
```

### 3. VWAPBreakout Strategy Class

Key parameters:
- `intraday_close_time`: Time to flatten positions (default: 15:45)
- `atr_stop`: ATR multiplier for trailing stop (default: 1.5)

Methods:
- `init()`: Initializes ATR indicator if stop loss is enabled
- `_atr()`: Static method for ATR calculation (14-period default)
- `next()`: Main strategy logic executed on each bar

## Usage

### Using Indian Stock Data (Recommended)

```python
# Fetch live data for Indian stocks
df = fetch_indian_stock_data("RELIANCE.NS", period="1y", interval="15m")

# Add VWAP calculation
df = add_vwap(df, time_col="Datetime")

# Run backtest with Indian market settings
bt = Backtest(
    df.iloc[500:2000],  # Data slice
    VWAPBreakout,
    cash=100_000,
    commission=0.001,  # 0.1% commission for Indian brokers
    exclusive_orders=True,
    trade_on_close=True
)

stats = bt.run()
print(stats)
bt.plot()
```

### Using CSV Data (Legacy)

```python
# Load and prepare data from CSV
df = pd.read_csv("your_data.csv")
df = add_vwap(df, time_col="Gmt time", dayfirst=True, format="%d.%m.%Y %H:%M:%S.%f")

# Run backtest
bt = Backtest(
    df[1000:5000],  # Data slice
    VWAPBreakout,
    cash=100_000,
    commission=0.000,
    exclusive_orders=True,
    trade_on_close=True
)

stats = bt.run()
print(stats)
bt.plot()
```

### Parameter Optimization

```python
# Optimize ATR stop multiplier (extended range for Indian markets)
stats_best, heatmap = bt.optimize(
    atr_stop=[round(x, 2) for x in np.arange(1, 3.01, 0.25)],
    maximize="Sharpe Ratio",
    return_heatmap=True
)

# Display comprehensive results
print(f"Best ATR Stop: {stats_best._strategy.atr_stop}")
print(f"Return: {stats_best['Return [%]']:.2f}%")
print(f"Sharpe Ratio: {stats_best['Sharpe Ratio']:.3f}")
print(f"Max Drawdown: {stats_best['Max. Drawdown [%]']:.2f}%")
```

## Data Sources

### Indian Stock Market Data (via yfinance)
**Recommended for live trading and analysis**

**Supported Symbols:**
- NSE stocks: Add ".NS" suffix (e.g., "RELIANCE.NS", "TCS.NS", "HDFCBANK.NS")
- BSE stocks: Add ".BO" suffix (e.g., "RELIANCE.BO")

**Popular Indian Stocks:**
- `RELIANCE.NS` - Reliance Industries (Default in notebook)
- `TCS.NS` - Tata Consultancy Services
- `HDFCBANK.NS` - HDFC Bank
- `INFY.NS` - Infosys
- `HINDUNILVR.NS` - Hindustan Unilever
- `ITC.NS` - ITC Limited
- `LT.NS` - Larsen & Toubro
- `KOTAKBANK.NS` - Kotak Mahindra Bank
- `BAJFINANCE.NS` - Bajaj Finance
- `MARUTI.NS` - Maruti Suzuki

**Timeframes:** 1m, 2m, 5m, 15m, 30m, 60m, 90m, 1h, 1d, 5d, 1wk, 1mo, 3mo
**Periods:** 1d, 5d, 1mo, 3mo, 6mo, 1y, 2y, 5y, 10y, ytd, max

### CSV Data Format (Legacy)
For custom data files:
- `Gmt time` or `Datetime`: Timestamp column
- `Open`: Opening price
- `High`: High price
- `Low`: Low price
- `Close`: Closing price
- `Volume`: Trading volume

Supported timeframes: Any intraday frequency (tested with 15-minute data)

## Performance Results

### Sample Results - TSLA (US Market)
Based on TSLA 15M data (June 2022 - June 2025):

| ATR Stop | Sharpe Ratio | Return (%) |
|----------|--------------|------------|
| 1.00     | 0.81         | 505.00     |
| 1.25     | 0.67         | 330.09     |
| **1.50** | **0.90**     | **714.45** |
| 1.75     | 0.79         | 496.07     |
| 2.00     | 0.72         | 371.97     |
| 2.25     | 0.64         | 282.63     |
| 2.50     | 0.62         | 266.77     |

**Optimal Parameters**: ATR Stop = 1.50 (Sharpe Ratio: 0.90, Return: 714%)

### Indian Market Considerations
- **Commission**: Strategy includes 0.1% commission for Indian brokers
- **Market Hours**: Adapt intraday close time for IST (Indian Standard Time)
- **Volatility**: Indian markets may require different ATR multipliers (1.0-3.0 range tested)
- **Liquidity**: Focus on highly liquid NSE stocks for best results
- **Data Quality**: yfinance provides reliable NSE data with automatic holiday filtering

**Recommended Indian Stocks for Testing:**
1. **RELIANCE.NS** - High liquidity, energy sector leader
2. **TCS.NS** - Stable IT stock with consistent volume
3. **HDFCBANK.NS** - Banking sector leader
4. **INFY.NS** - IT sector with good intraday movement
5. **BAJFINANCE.NS** - NBFC with high volatility
6. **MARUTI.NS** - Auto sector representative

## Dependencies

```
pandas
numpy
backtesting
matplotlib
yfinance
```

Install with:
```bash
pip install -r requirements.txt
```

**Note**: yfinance is required for fetching Indian stock market data.

## Key Considerations

### Strategy Strengths
- Mean reversion and momentum combination
- Risk management with trailing stops
- Intraday focus reduces overnight risk
- Parameter optimization capability

### Limitations
- Requires liquid instruments with reliable volume data
- Performance may vary across different market conditions
- Intraday strategy requires quality tick data
- yfinance data may have occasional gaps during market holidays
- Indian market data limited to yfinance availability

### Risk Warnings
- **Past performance does not guarantee future results**
- **Strategy requires proper risk management** - monitor drawdowns closely
- **Market conditions change** - regularly re-optimize parameters
- **Always test with paper trading** before live implementation
- **Indian market specific risks**: Consider market volatility, liquidity, and regulatory changes
- **Data dependency**: yfinance data quality may vary; validate before trading

## Getting Started

### Quick Start with Indian Stocks
1. **Install dependencies**: `pip install -r requirements.txt`
2. **Open notebook**: `jupyter notebook vwap_stocks.ipynb`
3. **Run all cells** - automatically fetches Reliance data and runs optimization
4. **View results** - comprehensive performance analysis with Indian market metrics
5. **Try other stocks** - modify `"RELIANCE.NS"` to test different companies
6. **Customize parameters** - adjust periods, intervals, and ATR multipliers

### Custom Setup
1. Choose your preferred Indian stock symbol (add .NS for NSE)
2. Select appropriate time period and interval
3. Run the VWAP calculation function
4. Execute the backtest with Indian market commission rates
5. Analyze results and optimize parameters
6. Validate with out-of-sample data before live trading

## Notebook Features

### Current Implementation
- ✅ **Automatic Data Fetching**: Gets latest Reliance data from NSE
- ✅ **Professional Reports**: Detailed performance analysis with risk metrics
- ✅ **Indian Market Optimization**: Extended ATR range (1.0-3.0) testing
- ✅ **Visual Analysis**: Interactive plots and comprehensive tables
- ✅ **Multi-Stock Support**: Easy switching between different NSE stocks
- ✅ **Risk Assessment**: Calmar ratio, win rate, and drawdown analysis
- ✅ **INR Formatting**: Portfolio values displayed in Indian Rupees

### Usage Examples in Notebook
```python
# Test different stocks
df = fetch_indian_stock_data("TCS.NS", period="6mo", interval="5m")
df = fetch_indian_stock_data("HDFCBANK.NS", period="2y", interval="1h")
```

## Future Enhancements

- Multi-timeframe VWAP analysis
- Additional technical indicators integration
- Portfolio-level backtesting with multiple Indian stocks
- Risk metrics enhancement
- Real-time trading implementation with Indian brokers
- **Indian Market Specific:**
  - IST timezone handling
  - Indian market holiday calendar integration
  - Sector-wise analysis (IT, Banking, Pharma, etc.)
  - Currency hedging for international portfolios
  - Integration with Indian brokerage APIs (Zerodha, Upstox, etc.)