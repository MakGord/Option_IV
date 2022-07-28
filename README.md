# Price Probabilty Plot

**Project Goal:** 

This project aims to create a statistical model that allows to visually estimate a potential level of magnitutde and direction of the price movement for a given security over a specified interval of time based on the historical data. 

**Model Inputs:** 
- Ticker Symbol
- Start Date for a Historical Interval
- End Date for a Historical Interval (Today's Date by default)
- Desired Time Frame (1 day, 1 week, 1 month)
- Price Resolution (Rounding base)

**Model Outputs:** 
- A Price Frequency table that counts the amount of time a stock has spent at each price point based on historical Open-High-Low-Close (OHLC) data. 

    [price_frequency_XBI_2010-01-01_2022-07-28.csv](https://github.com/MakGord/Price_Probability_Plot/blob/main/price_frequency_XBI_2010-01-01_2022-07-28.csv)

- A Price Probability Plot that overlays the Price Frequency data with a normal distribution plot calculated at the last available price (as mu) and historical standard deviation (as sigma). 

<img src="price_probability_plot_XBI_2010-01-01_2022-07-28.png?raw=true"/>

**Code:** 


[price_probability_plot.ipynb](https://github.com/MakGord/Price_Probability_Plot/blob/main/price_probability_plot.ipynb)

[![Binder](http://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/MakGord/Price_Probability_Plot/main?filepath=index.ipynb)

1. Import Modules.
```
import math
import numpy as np
import pandas as pd
import yfinance as yf
from scipy import stats
from datetime import date
import plotly.graph_objects as go
import matplotlib.pyplot as plt
import matplotlib.ticker as mtick
```
2. Initialize Variables. 
```
ticker="XBI"
start_date="2010-01-01"
end_date=date.today()
time_interval="1wk"
rounding_base=1
```

3. Download Historical Data from Yahoo Finance
```
def get_stock_data(ticker, start_date, end_date, time_interval):
    df = yf.download(ticker, start_date, end_date, interval = time_interval).dropna()
    return df
```


4. Extract the Last Closing Price
```
def get_last_price(df):
    last_price = df.Close[len(df.Close) - 1]
    return last_price
```

5. Calcualate Standard Deviation of the Log Return.
```
def get_st_dev(df):
    df["log_r"] = np.log(df.Close) - np.log(df.Close.shift(1))
    st_dev_pct = np.std(df.log_r)
    return st_dev_pct
```
6. Round Historical Data.
```
def round_data(df, rounding_base):
    df = roinding_base * round(df / rounding_base)
    return df
```

7. Group Historical Data by counting the sum of Opens, High, Lows and Closes (OHLC) for each Price Point
```
def group_data(df):
    df["Date"] = df.index

    # Individually group OHLC columns
    df_group_open = pd.DataFrame(
        df.groupby("Open").count().sort_values(by="Date", ascending=False).Date
    )
    df_group_high = pd.DataFrame(
        df.groupby("High").count().sort_values(by="Date", ascending=False).Date
    )
    df_group_low = pd.DataFrame(
        df.groupby("Low").count().sort_values(by="Date", ascending=False).Date
    )
    df_group_close = pd.DataFrame(
        df.groupby("Close").count().sort_values(by="Date", ascending=False).Date
    )

    # Join OHLC columns
    df_joined = df_group_open.join(df_group_high, how="outer", rsuffix="o", sort=True)
    df_joined = df_joined.join(df_group_low, how="outer", rsuffix="h", sort=True)
    df_joined = df_joined.join(df_group_close, how="outer", rsuffix="l", sort=True).fillna(0)
    # Rename columns and index
    df_joined.columns = ["count_open", "count_high", "count_low", "count_close"]
    df_joined.index.name = "dollar_price_point"
    # Add sum column
    df_joined["count_sum"] = df_joined.sum(axis=1)
    # Add percentage sum column
    df_joined["count_sum_pct"] = df_joined.count_sum / np.sum(df_joined.count_sum)
    # Sort by percentage sum
    df_joined = df_joined.sort_values(by="count_sum_pct", ascending=False)
    return df_joined
```

8. Plot the Percentage Sum Data from 7.

```
def plot_data(df, ticker, start_date, end_date, rounding_base):
    fig, ax = plt.subplots()
    # Set X and Y
    X = df.index
    Y = df.count_sum_pct
    # Create Bar Plot
    ax.bar(X, Y, alpha=0.7)
    # Add Grid
    ax.grid()
    # Resize Plot
    fig.set_size_inches(56, 16)
    # Add X-ticks
    ax.xaxis.set_ticks(np.arange(np.min(X), np.max(X), rounding_base))
    # Change Y-axis format to percent
    ax.yaxis.set_major_formatter(
        mtick.PercentFormatter(xmax=1, decimals=None, symbol="%", is_latex=False)
    )
    # Set X-label
    ax.set_xlabel("Price Point ($)", size=15)
    # Set Y-label
    ax.set_ylabel('Time (%) spent at the Price Point', size=15)
    # Set Title
    ax.set_title(
        "Price Probability Plot for: "
        + ticker
        + ", from = "
        + str(start_date)
        + " to "
        + str(end_date)
        + ", at interval =  "
        + time_interval,
        size=30,
    )
    # Add data labels
    for x, y in zip(X, Y):
        label = "${:.0f}".format(x)
        plt.annotate(
            label,  
            (x, y),  
            textcoords="offset points",  
            xytext=(-10, 15),  
            fontsize=8,
            ha="left",
        )  
    # Set axes limits
    ax.set_xlim(np.min(X) * 0.95, np.max(X) + np.min(X) * 0.05)
    ax.set_ylim(np.min(Y) * 0.95, np.max(Y) * 1.05)
    return (fig, ax)
```
9. Add Normal Distribtuion to the plot. 
```
def plot_normal(fig, ax, last_price, st_dev_pct):
    # Add second Y-axis
    ax1 = ax.twinx()
    # Plot the bell-curve
    mu = last_price
    st_dev_dollar = st_dev_pct * last_price
    snd = stats.norm(mu, st_dev_dollar)
    x = np.linspace(0, 100, 1000)
    y = snd.pdf(x)
    ax1.plot(x, y, "--r")
    ax1.set_ylim(0, 0.75)

    # Plot Stadnard Deviation Points
    sigmas = [
        mu,
        mu + st_dev_dollar,
        mu + st_dev_dollar * 2,
        mu - st_dev_dollar,
        mu - st_dev_dollar * 2,
    ]

    heights = [
        np.max(y),
        np.max(y) / 1.6,
        np.max(y) / 7,
        np.max(y) / 1.6,
        np.max(y) / 7,
    ]

    for x, y in zip(sigmas, heights):
        ax1.plot([x, x], [0, y], "--*r", markersize=12)

        plt.annotate(
            "${:.2f}".format(x),
            (x, y),
            textcoords="offset points",
            xytext=(-10, 15),
            fontsize=8,
            ha="left",
        )

    return ax.plot
 ```


