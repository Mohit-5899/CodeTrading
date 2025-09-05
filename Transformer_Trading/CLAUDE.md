# Transformer Trading Project

## Project Overview
This project implements a Transformer-based neural network for forex price prediction using EUR/USD hourly candlestick data. The model uses technical indicators and historical price data to forecast future closing prices.

## Architecture
- **Model**: TimeSeriesTransformer with encoder layers
- **Input Features**: 9 dimensions (OHLC + technical indicators)
- **Sequence Length**: 30 timesteps
- **Prediction**: Next 1 candlestick close price
- **Technical Indicators**: RSI, Bollinger Bands, Moving Average Slope

## Key Components

### Data Processing (`Transformer_Trading.ipynb`)
- **Data Loading**: `load_forex_data()` - loads CSV with GMT timestamps
- **Feature Engineering**: `add_technical_indicators()` - adds RSI, Bollinger Bands, MA slope
- **Scaling**: `select_and_scale_features()` - MinMax normalization
- **Dataset**: `ForexDataset` class for sequence generation

### Model Architecture
```
Input: (Batch, 30, 9)
↓
Linear: 9 → 64
↓
+Positional Embedding (1, 30, 64)
↓
Transformer Encoder (2 layers, 8 heads, FF=256)
↓
Output Linear: 64 → 1
↓
Predictions: (Batch, 1)
```

### Training Configuration
- **Optimizer**: Adam (lr=1e-3)
- **Loss Function**: MSE Loss
- **Epochs**: 20
- **Batch Size**: 32
- **Data Split**: 80% train, 10% validation, 10% test
- **Device**: CUDA if available, else CPU

## Dependencies
```bash
pip install torch pandas numpy scikit-learn ta matplotlib
```

## File Structure
```
CodeTrading/
├── Transformer_Trading.ipynb    # Main implementation
├── EURUSD_Candlestick_1_Hour_BID_01.07.2020-15.07.2023.csv  # Data file
└── CLAUDE.md                    # This documentation
```

## Usage Instructions

### 1. Data Preparation
Ensure your CSV file has these columns:
- `Gmt time`: DateTime format '%d.%m.%Y %H:%M:%S.%f'
- `Open`, `High`, `Low`, `Close`: Price values

### 2. Running the Model
Execute all cells in `Transformer_Trading.ipynb` sequentially:
1. Install dependencies
2. Load and process data
3. Create datasets and dataloaders
4. Initialize and train model
5. Evaluate results

### 3. Key Parameters to Modify
- `seq_length`: Input sequence length (default: 30)
- `pred_length`: Prediction horizon (default: 1)
- `feature_cols`: Features to include in training
- `epochs`: Training iterations (default: 20)
- `lr`: Learning rate (default: 1e-3)

## Model Performance
The trained model achieves low MSE and MAE on test data. Use `evaluate_model()` to:
- Calculate MSE and MAE metrics
- Plot real vs predicted prices
- Visualize prediction accuracy over time windows

## Technical Indicators Used
1. **RSI (14-period)**: Momentum oscillator
2. **Bollinger Bands**: Volatility bands (20-period, 2 std dev)
3. **MA Slope**: 20-period moving average slope for trend detection

## Customization Options
- **Multi-step Prediction**: Modify `pred_length` for forecasting multiple candles
- **Feature Selection**: Adjust `feature_cols` to include/exclude indicators
- **Model Architecture**: Modify layers, heads, dimensions in `TimeSeriesTransformer`
- **Time Frames**: Change data granularity by using different CSV files

## Training Commands
No specific CLI commands - run Jupyter notebook cells interactively.

## Testing
Use the `evaluate_model()` function with different parameters:
- `window_width`: Number of predictions to plot
- `start_index`: Starting point in test dataset
- Visual comparison of real vs predicted prices

## Notes
- Uses sequential train/val/test split (no random shuffling) to preserve temporal order
- MinMax scaling applied to all features
- Positional embeddings learned during training
- Model outputs require inverse scaling for actual price values