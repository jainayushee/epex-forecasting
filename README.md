# Project: European Power Exchange (EPEX) forecasting

Github: https://github.com/jainayushee/epex-forecasting

## Overview

The objective of this project is to forecast electricity market prices and volumes at a 15-minute interval on the European Power Exchange (EPEX), with a forecasting horizon of 10 time steps. The task involves providing predictions for all assets and their "High," "Low," "Close," and "Volume" at each time point, using pre-trained models or time series libraries, and evaluating performance using the Symmetric Mean Absolute Percentage Error (sMAPE).

Several methods were tested to optimize the memory usage and processing time. However, due to hardware limitations, only certain methods provided partial success.

## Methods Tried

1. **Float16 Conversion**
   - The first optimization attempt was to convert numerical columns to `float16` to reduce memory usage. This method provided some improvement, but it was not enough to handle the full dataset efficiently on the existing hardware.

2. **Out-of-Core Processing with Dask**
   - The second approach involved using the out-of-core library `Dask` to parallelize the processing and distribute it across multiple workers. However, an optimal state was never achieved due to memory overload. The workers frequently crashed, making this approach infeasible for the given hardware.

3. **Subsampling the Data**
   - Finally, the dataset was subsampled to fit within the constraints of the existing hardware. This also allowed the processing to complete successfully only partially.

### Methodology

1. Data partitioning

To prevent data leakage, it is essential to split the data while preserving the temporal sequence in which the values were observed, rather than using a random split. Given the constraints, the training data was limited to nine months, the validation data covered three months in 2023, and the testing data is from 2024.

2. Feature Engineering

The initial plan for feature engineering was to use Autocorrelation Function (ACF) and Partial Autocorrelation Function (PACF) plots to determine the optimal number of lags. ACF and PACF plots are typically used to assess how past observations influence future values in a time series. ACF measures the overall correlation between the time series and its lagged versions, while PACF focuses on the direct influence of specific lags after accounting for the effects of intermediate lags. 

However, due to the large volume of data, it wasn't feasible to generate these plots. As a result, a decision was made to use 10 lags, which seemed like a reasonable choice based on the problem statement.

3. Model Selection (multivariate time series data)

LSTM (Long Short-Term Memory) models are specialized for handling sequential data, particularly univariate time series where each data point represents a single variable over time. In the context of financial assets, each asset exhibits unique temporal patterns and statistical properties. Therefore, to accurately capture these individual dynamics, LSTM models need to be trained separately for each asset. Training a single LSTM model on combined data from multiple assets can lead to confusion within the model, as it may struggle to differentiate and learn the distinct patterns of each asset, resulting in diminished predictive performance.

On the other hand, N-BEATS (Neural Basis Expansion Analysis for Interpretable Time Series Forecasting) is a deep learning architecture designed to handle both univariate and multivariate time series data. N-BEATS was tried because it can model multiple assets simultaneously, capturing shared patterns and interdependencies across different time series. This multivariate capability allows N-BEATS to leverage information from multiple assets to potentially improve forecasting accuracy. By modeling the collective behavior of assets, N-BEATS can identify common trends and seasonality that individual univariate models like LSTM might miss when trained separately.