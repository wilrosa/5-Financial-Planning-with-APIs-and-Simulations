# Financial-Planning-with-APIs-and-Simulations

This repo contains the results of the module 5 challenge. I assumed the role of a financial consultant that focuses on projects to benefit local communities. This project entails building a tool to help members of a local credit union evaluate their financial health. 

Specifically, the credit union board wants the members to be able to do two things. First, they should be able to assess their monthly budgets. Second, they should be able to forecast a reasonably effective retirement plan based on their current holdings of cryptocurrencies, stocks, and bonds. I created two financial analysis tools using a single Jupyter notebook:

(1) A financial planner for emergencies. The members are now able to use this tool to visualize their current savings. The members can  determine if they have enough reserves for an emergency fund.

(2) A financial planner for retirement. This tool forecasts the performance of their retirement portfolio in both 10 and 30 years. To do this, the tool makes an Alpaca API call via the Alpaca SDK to get historical price data for use in Monte Carlo simulations.

I used the information from the Monte Carlo simulations to answer questions about each portfolio.

---

## Technologies

This project leverages python 3.7 with the following libraries and dependencies:

* [pandas](https://github.com/pandas-dev/pandas) - For manipulating data

* [matplotlib](https://github.com/matplotlib/matplotlib) - For creating static, animated, and interactive visualizations

* [os](https://docs.python.org/3/library/os.html) - Portable way of using operating system dependent functionality

* [requests](https://github.com/psf/requests) - For easily sending HTTP/1.1 requests 

* [json](https://www.json.org/json-en.html) - lightweight data-interchange format

* [dotenv](https://github.com/theskumar/python-dotenv) - For reading key-value pairs from a .env file 

* [alpaca_trade_api](https://github.com/alpacahq/alpaca-trade-api-python) - for the Alpaca Commission Free Trading API

---

## Installation Guide

MCForecastTools.py provides a python class for runnning Monte Carlo simulations on portfolio price data. We imported the `MCSimulation` function that Constructs all the necessary attributes for the MCSimulation object.

We also use the `load_dotenv` funtion to import the necessary API keys.

```python

from MCForecastTools import MCSimulation
load_dotenv("api_keys")

```
---

## **Part 1: Create a Financial Planner for Emergencies**

### **Evaluate the Cryptocurrency Wallet by Using the Requests Library**

In this section, we determine the current value of a member’s cryptocurrency wallet. Here, we collect the current prices for the Bitcoin and Ethereum cryptocurrencies by using the Python Requests library.

```python
btc_url = "https://api.alternative.me/v2/ticker/Bitcoin/?convert=USD"
eth_url = "https://api.alternative.me/v2/ticker/Ethereum/?convert=USD"

btc_response = requests.get(btc_url).json()
eth_response = requests.get(eth_url).json()

print(json.dumps(eth_response, sort_keys=True, indent=3))
print(json.dumps(btc_response, sort_keys=True, indent=3))
```

### **Evaluate the Stock and Bond Holdings by Using the Alpaca SDK**

In this section, we determine the current value of a member’s stock and bond holdings. We make an API call to Alpaca via the Alpaca SDK to get the current closing prices of the SPDR S&P 500 ETF Trust (ticker: SPY) and of the iShares Core US Aggregate Bond ETF (ticker: AGG). For the prototype, we assumed that the member holds 110 shares of SPY, which represents the stock portion of their portfolio, and 200 shares of AGG, which represents the bond portion.

```python
alpaca_api_key = os.getenv("ALPACA_API_KEY")
alpaca_secret_key = os.getenv("ALPACA_SECRET_KEY")

display(type(alpaca_api_key))
display(type(alpaca_secret_key))

alpaca = tradeapi.REST(
    alpaca_api_key,
    alpaca_secret_key,
    api_version="v2")

prices_df = alpaca.get_bars(
    tickers,
    timeframe,
    start=start_date,
    end=end_date
).df

SPY = prices_df[prices_df['symbol']=='SPY'].drop('symbol', axis=1)
AGG = prices_df[prices_df['symbol']=='AGG'].drop('symbol', axis=1)

prices_df = pd.concat([SPY, AGG], axis=1, keys=["SPY", "AGG"])

prices_df.head()
```

### **Evaluate the Emergency Fund**

In this section, we use the valuations for the cryptocurrency wallet and for the stock and bond portions of the portfolio to determine if the credit union member has enough savings to build an emergency fund into their financial plan.

```python
if total_portfolio > emergency_fund_value:
    print(f"Congratulations! You have enough money to fund your emergency portfolio.")
elif total_portfolio == emergency_fund_value:
    print(f"Congratulations on reaching this important financial goal!")
else:
    print(f"You're almost there! You are only ${emergency_fund_value - total_portfolio} from reaching your goal.")
```

## **Part 2: Create a Financial Planner for Retirement**

### **Create the Monte Carlo Simulation**

In this section, we use the MCForecastTools library to create a Monte Carlo simulation for the member’s savings portfolio.

```python
prices_df_3yr = alpaca.get_bars(
    tickers,
    timeframe,
    start=start_date_sim,
    end=end_date_sim
).df

MC_60_40 = MCSimulation(
    portfolio_data = prices_df_3yr,
    weights = [.60,.40],
    num_simulation = 500,
    num_trading_days = 252*30
)
```

### **Analyze the Retirement Portfolio 30-year Forecasts**

Using the current value of only the stock and bond portion of the member's portfolio and the summary statistics that we generated from the Monte Carlo simulation, we determined the range of returns of potential investment returns.

```python
print(f"There is a 95% chance that the current balance of the stock and bond portion of the member's portfolio, which is ${total_stocks_bonds},"
      f" will end within in the range of"
      f" ${ci_lower_thirty_cumulative_return} and ${ci_upper_thirty_cumulative_return} with a split of 40% to AGG and 60% to SPY over the next 30 years.")
```

### **Forecast Cumulative Returns in 10 Years**

Here, we adjust the retirement portfolio and run a new Monte Carlo simulation to find out if adjusting the weights of the retirement portfolio so that the composition for the Monte Carlo simulation consists of 20% bonds and 80% stocks will allow members to retire after 10 years.

```python
MC_80_20 = MCSimulation(
    portfolio_data = prices_df_3yr,
    weights = [.80,.20],
    num_simulation = 500,
    num_trading_days = 252*10
)
```

* ![MC_Simulation_Line_Plot](/Images/MC_Simulation_Line_Plot.png)

---
## Contributors

Brought to you by Wilson Rosa. https://www.linkedin.com/in/wilson-rosa-angeles/.

---
## License

MIT
