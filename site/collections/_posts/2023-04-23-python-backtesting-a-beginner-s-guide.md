---
date: 2023-04-23 11:00:00
title: 'Python Backtesting: A Beginner''s Guide'
description: >-
  We explore the use of historical crypto price data to simulate buy and sell
  signals for various simple trading strategies using Python.
tags:
  - python
  - automated-trading
  - crypto
  - aws
  - s3
image: /uploads/backtesting.jpeg
---
# Python Backtesting: A Beginner's Guide

In my [last post](https://zakraicik.xyz/blog/cyrpto-prices-in-python){: target="_blank" rel="noopener"}, I wrote about to use the [Binance](https://www.binance.us/){: target="_blank" rel="noopener"} API to pull historical prices for various Crypto assets. In this article, we will start to build the foundation for&nbsp;**simple** backtester. A backtester is a software tool that allows traders and investors to simulate the performance of their trading strategies using historical market data. By applying a set of rules to the historical data, a backtester can provide valuable insights into how a strategy would have performed in the past, helping traders make more informed decisions in the future.

We will focus on generating buy/sell/hold signals against historical price data using various trading strategies. Simulating trades, associated costs, performance etc. is out of scope for this post but will be covered in a future post. Specifically, we will cover the following:

* Homegrown backtester vs. pre-built backtester
* Simulating buy/sell signals using various strategies
* Next Steps

The full repository can be found [here](https://github.com/zakraicik/crypto_trader/tree/main/crypto_trader){: target="_blank" rel="noopener"}. Keep in mind this article covers one part of a bigger project.&nbsp;

## Homegrown or Pre-built?

---

There are a lot of prebuilt backtesters available in Python already including Backtrader, PyAlgoTrade, Zipline, and Catalyst. Knowing that so many backtesters are already built, why would we build our own?&nbsp;

### **Customization**

First, when constructing your backtester, you have the opportunity to handpick the features that are most relevant to your investment approach. This means you can include elements such as custom trading signals, advanced risk management options, and various asset classes, allowing for a more comprehensive and nuanced analysis.

Additionally, by developing your own backtester, you can integrate multiple data sources, ensuring that your model has access to the most relevant and up-to-date information. This allows for more accurate backtesting, which can lead to better decision-making when it comes to implementing your strategies in the real world.

Another advantage of having a custom-built backtester is the ability to incorporate your own unique algorithms and trading strategies. This can be especially valuable for quantitative traders who require highly specialized tools to execute their complex models. A tailored backtester can accommodate these advanced strategies and test them with precision, offering insights that may not be possible with off-the-shelf software.

### **Flexibility**

Modifying and adapting your custom-built backtester as your trading strategies evolve is a powerful advantage that provides you with the ability to stay ahead in the ever-changing financial markets. This dynamic approach not only gives you greater flexibility and control over your investment decisions but also allows you to continually refine and improve your strategies, leading to enhanced performance and risk management.

One of the primary benefits of having an adaptable backtester is that it enables you to incorporate new data sources, analytical techniques, and market insights as they become available. Financial markets are constantly evolving, with new information and trends emerging regularly. An adaptable backtester ensures that you can seamlessly integrate these changes, ensuring that your trading strategies remain up-to-date and relevant.

Furthermore, as you gain experience and insights from trading, you may develop new ideas or hypotheses about market behavior that you want to test. With a customizable backtester, you can quickly and easily incorporate these new concepts and test their effectiveness. This ability to experiment and iterate on your trading strategies can lead to the discovery of profitable opportunities that might have otherwise been overlooked.

### **Cost**&nbsp;

Building your own backtester can indeed be a more cost-effective solution in the long run as compared to using pre-existing or subscription-based backtesting software. This advantage becomes even more significant when you plan on utilizing the backtester over an extended period of time or have specialized needs that may require frequent updates or customizations.

There are several reasons why creating your own backtester can be more economical in the long run:

1. **Initial investment:** While developing your own backtester may require an upfront investment in terms of time and resources, this cost is typically a one-time expense. On the other hand, pre-existing solutions often come with ongoing subscription fees or recurring charges, which can add up significantly over time.
2. **Customization costs:** Pre-existing backtesting platforms may charge additional fees for customizations or advanced features. Building your own backtester allows you to include these features from the outset, without incurring extra expenses. Furthermore, as your needs evolve, you can implement modifications yourself, avoiding costly updates or service charges from third-party providers.
3. **Scalability:** A custom-built backtester is designed specifically to accommodate your unique trading strategies and requirements. As your trading volume or complexity increases, your backtester can be scaled up accordingly, without the need for costly upgrades or additional subscriptions that might be necessary with pre-existing solutions.
4. **Independence:** By developing your own backtester, you become self-sufficient and are not reliant on the continued support or availability of a third-party platform. This can help you avoid unexpected disruptions, such as a platform shutting down or experiencing technical issues, which could result in additional costs or delays in your trading activities.
5. **Intellectual property:** When you create your own backtester, you retain full ownership of the software and its underlying algorithms, giving you the freedom to use, modify, or even sell the system as you see fit. In contrast, pre-existing solutions often come with licensing restrictions or usage limitations that could constrain your flexibility and potentially generate additional expenses in the future.

### **Learning**

Building your own backtester is an invaluable learning experience that can significantly deepen your understanding of how backtesting systems work, as well as the underlying mechanics of trading strategies and financial markets. By undertaking the development process yourself, you gain hands-on experience and insight into the intricacies of backtesting, which can ultimately lead to better trading decisions and improved investment performance.

Here are several ways that constructing your own backtester enhances your learning:

1. **Understanding the data:** In order to create an effective backtester, you need to be well-versed in handling various types of market data, such as historical price, volume, and fundamental information. This process helps you develop a deeper appreciation of data quality, data cleaning, and data management techniques, all of which are crucial in generating accurate backtesting results.
2. **Developing trading algorithms:** Building a backtester requires you to translate your trading ideas into specific rules and algorithms. This step not only helps you refine and clarify your strategies but also teaches you about the challenges and limitations associated with algorithmic trading, such as overfitting, curve-fitting, and lookahead bias.
3. **Implementing risk management:** An integral aspect of creating a backtester is incorporating risk management strategies to evaluate and control the potential losses in your trading systems. By designing these mechanisms yourself, you gain a better understanding of the importance of risk management and the various methods used to mitigate risk, such as position sizing, stop-loss orders, and diversification.
4. **Evaluating performance:** A well-designed backtester includes various performance metrics to assess the effectiveness of your trading strategies. By implementing these metrics, you can gain insights into the strengths and weaknesses of your strategies and learn how to interpret and analyze these results to improve your trading decisions.
5. **Simulation and optimization:** Building your own backtester enables you to experiment with different market conditions, parameter values, and timeframes. This hands-on experience allows you to learn about the robustness of your trading strategies, the importance of out-of-sample testing, and the dangers of over-optimization.
6. **Debugging and troubleshooting:** Developing a backtester often involves encountering and solving various issues related to coding, logic, or data. Troubleshooting these problems not only enhances your programming skills but also helps you learn about the common pitfalls and challenges associated with backtesting.

##

## Simulating buy/sell signals using various strategies

---

As mentioned at the start of the article- the focus of this post is using various trading strategies to generate buy/sell signals against historical price data. This is the core functionality required to build a simple backtester, as once we know when to buy/sell, we can layer profit and risk calculations.&nbsp;

### Helper.py

A helper file is typically used to store helper functions or utility functions that are used throughout an application or project. These functions are designed to perform specific tasks and can be reused across different parts of the application. The completed helper can be found&nbsp;[here](https://github.com/zakraicik/crypto_trader/blob/main/crypto_trader/helper.py){: target="_blank" rel="noopener"}. In this section, we expand the `helper.py` file we created when using the [Binance API to pull historical crypto prices](https://zakraicik.xyz/blog/cyrpto-prices-in-python){: target="_blank" rel="noopener"}. At this point, we assume you have working Binance and AWS keys. If you don't, please refer [here](https://zakraicik.xyz/blog/cyrpto-prices-in-python){: target="_blank" rel="noopener"}.

The first thing we added to our `helper.py`&nbsp;is a function used to pull down our historical price data from S3 and store it in a pandas DataFrame. The function takes four input parameters:

1. A string `bucket` representing the name of an Amazon S3 bucket
2. A string `key` representing the object key within the bucket
3. Two strings, `access_key_id` and `secret_access_key`, representing AWS security credentials.&nbsp;

Inside the function, it initializes a boto3 S3 client using the provided security credentials. Then, it retrieves the specified object from the S3 bucket using the `get_object` method, which is stored in the `obj` variable. The content of the object is read from the body, decoded from utf-8 to a string, and stored in the `data` variable.

Next, the function creates a pandas DataFrame named `df` from the JSON data contained in the `data` variable, with the data orientation set to "records". Following this, two columns in the DataFrame, "open\_time" and "close\_time", are converted to datetime objects with milliseconds as the unit of time. Finally, the resulting DataFrame is returned to the caller.

```python
def from_s3(
    bucket: str, key: str, access_key_id: str, secret_access_key: str
) -> pd.DataFrame:
    s3 = boto3.client(
        "s3", aws_access_key_id=access_key_id, aws_secret_access_key=secret_access_key
    )
    obj = s3.get_object(Bucket=bucket, Key=key)
    data = obj["Body"].read().decode("utf-8")
    df = pd.read_json(data, orient="records")
    df["open_time"] = pd.to_datetime(df["open_time"], unit="ms")
    df["close_time"] = pd.to_datetime(df["close_time"], unit="ms")
    return df
```

The only other change we made to helper.py is a function that can be used to plot buy/sell signals against historical price data. The function takes two pandas DataFrame objects as input parameters, `data` and signals.&nbsp;

1. `data` is a DataFrame containing our historical price data from AWS
2. `signals` is a DataFrame containing buy/sell/hold indicators for every data in our data DataFrame.

Inside the function, the `data` and `signals` DataFrames are merged into a new DataFrame called `data_signals`, based on matching 'open\_time' values from `data` with 'timestamp' values from `signals`. The merge operation is performed with a left join.

The function then sets up a plot using the `matplotlib.pyplot` library, which is imported as `plt`. The plot has a size of 14x8 inches. The close prices, short-term SMA, and long-term SMA are plotted as lines on the graph with different labels.

Buy and Sell signals are extracted from the `data_signals` DataFrame based on the 'position' column values (1 for Buy, -1 for Sell) and are overlaid on the graph as green upward-pointing triangles and red downward-pointing triangles, respectively.

The plot is then labeled with axis labels ('Timestamp' and 'Price') and a legend, and given a title, "Price data with Buy/Sell signals and SMAs". Finally, the function displays the plot using the `plt.show()` method.

```python
def plot_signals(data: pd.DataFrame, signals: pd.DataFrame):
    # Merge price data and signals
    data_signals = pd.merge(
        data, signals, left_on="open_time", right_on="timestamp", how="left"
    )

    # Plot the close price
    plt.figure(figsize=(14, 8))
    plt.plot(
        data_signals["timestamp"], data_signals["close"], label="Price", linewidth=1
    )

    # Plot short SMA
    plt.plot(
        data_signals["timestamp"],
        data_signals["short_sma"],
        label=f"Short SMA",
        linewidth=1,
    )

    # Plot long SMA
    plt.plot(
        data_signals["timestamp"],
        data_signals["long_sma"],
        label=f"Long SMA",
        linewidth=1,
    )

    # Overlay buy signals
    buy_signals = data_signals[data_signals["position"] == 1]
    plt.scatter(
        buy_signals["timestamp"],
        buy_signals["close"],
        label="Buy",
        marker="^",
        color="g",
    )

    # Overlay sell signals
    sell_signals = data_signals[data_signals["position"] == -1]
    plt.scatter(
        sell_signals["timestamp"],
        sell_signals["close"],
        label="Sell",
        marker="v",
        color="r",
    )

    # Add labels and legend
    plt.xlabel("Timestamp")
    plt.ylabel("Price")
    plt.legend(loc="best")
    plt.title("Price data with Buy/Sell signals and SMAs")

    # Display the plot
    plt.show()
```

### Strategies.py

You can think of `strategies.py` as a repository of all the various trading strategies you are in interested in testing against historical prices. For the purpose of this post, we will focus on a very simple strategy called the SMA crossover strategy. However, you can store as many strategies as you would like in this file.&nbsp;

The Simple Moving Average (SMA) crossover strategy is a popular technical analysis-based trading strategy that relies on moving averages to generate buy and sell signals. It involves the use of two SMAs: a short-term SMA and a long-term SMA. The short-term SMA is calculated over a shorter period (e.g., 10, 20, or 50 days) and the long-term SMA is calculated over a longer period (e.g., 100, 200, or 500 days). The main idea behind the strategy is to identify trend reversals and trade accordingly.

When the short-term SMA crosses above the long-term SMA, it is considered a bullish signal (a Buy signal), suggesting that the asset's price is gaining upward momentum and may continue to rise. Conversely, when the short-term SMA crosses below the long-term SMA, it is considered a bearish signal (a Sell signal), indicating that the asset's price is losing upward momentum and may continue to decline.

In our strategies.py file, we define a function sma\_crossover\_strategy&nbsp; that takes a pandas DataFrame `data` and two integers `short_window` and `long_window` as input parameters. The function is designed to generate trading signals based on the SMA crossover strategy using the given short-term and long-term SMA window lengths.

Inside the function, two SMAs, `short_sma` and `long_sma`, are calculated using the `calculate_sma` function, which takes `data` and the respective window lengths as its arguments. A new DataFrame, `signals`, is created with the same index as `data`. The 'timestamp' column in the `signals` DataFrame is populated with the 'open\_time' values from the `data` DataFrame.

Initially, the 'position' column in the `signals` DataFrame is set to zero for all rows. Then, starting from the `long_window` row, the 'position' column is set to 1 if the `short_sma` is greater than the `long_sma`, and to -1 otherwise. The 'position' column is then updated with the difference between consecutive position values, which generates Buy (1) and Sell (-1) signals.

Next, the 'short\_sma' and 'long\_sma' columns in the `signals` DataFrame are populated with the calculated SMA values. The 'position' column is then updated one more time to remove any remaining zero values, setting them to either 1 or -1 based on the previous conditions.

Finally, the function returns the `signals` DataFrame with the index reset. The output DataFrame contains columns for 'timestamp', 'position', 'short\_sma', and 'long\_sma', which can be used to visualize and analyze the SMA crossover strategy signals. A `position` &gt; 0 indicates a buy, while a `position` &lt; 0 indicates a sell.&nbsp;

```python
def sma_crossover_strategy(
    data: pd.DataFrame, short_window: int, long_window: int
) -> pd.DataFrame:
    short_sma = calculate_sma(data, short_window)
    long_sma = calculate_sma(data, long_window)

    signals = pd.DataFrame(index=data.index)

    signals["timestamp"] = data["open_time"]

    signals["position"] = 0

    signals["position"][long_window:] = np.where(
        short_sma[long_window:] > long_sma[long_window:], 1.0, -1.0
    )

    signals["position"] = signals["position"].diff()

    signals["short_sma"] = short_sma

    signals["long_sma"] = long_sma

    signals["position"] = np.where(
        signals["position"] < 0, -1, np.where(signals["position"] > 0, 1, 0)
    )

    return signals.reset_index(drop=True)
```

### Simulate.py

At this point, we can use the functions in `helper.py` and `strategy.py` to pull down our historical prices and generate buy/sell signals using the strategy of interest. Please note, for the purposes of this post - some arguments required by various functions are hard coded. To "productionalize" this code you could make the arguments dynamic.&nbsp;

`simulate.py` is a very short file which uses from\_s3 to pull our historical price data, generates buy/sell signals based on that data, and plots the signals against the price data. `short_window` and `long_window` are the number of data points to consider in our short and long SMA.&nbsp;

```python
def main():
    s3_key = "data/ETHUSDT/2017_04_15_2023_04_10_1d.json"

    data = from_s3(BUCKET, s3_key, AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY)

    short_window = 50
    long_window = 200

    signals = sma_crossover_strategy(data, short_window, long_window)

    plot_signals(data, signals)
```

The generated plot would look something like this.&nbsp;

![](/uploads/screenshot-2023-04-23-at-7-40-50-am.png){: width="2364" height="1376"}

## Next Steps

---

We have built a framework we can use to generate buy/sell/hold signals from various trading strategies. However, we are far from a complete backtester. Some high level next steps in the evolution of our homegrown backtester are listed below.&nbsp;

1. **Simulate trades:** With your trading strategy implemented, simulate trades based on the buy/sell/hold signals generated. Track the performance of your trades, including the profit/loss on each trade and the overall portfolio performance.
2. **Evaluate performance:** Once you have simulated trades, evaluate the performance of your trading strategy. This includes calculating metrics such as Sharpe ratio, maximum drawdown, and other performance indicators. Compare the performance of your trading strategy against a benchmark, such as the S&P 500, to see how it stacks up.

Feel free to reach out with any questions using the&nbsp;[contact](https://zakraicik.xyz/contact/)&nbsp;page or hitting me up on any of my social links!