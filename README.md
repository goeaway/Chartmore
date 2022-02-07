# Chartmore

This repo illustrates a bit about the Chartmore algorithmic trading bot system. We started this project with the intention of building an application that can autonomously trade multiple different financial instruments at once, with minimal interaction from us!

The system is made up of 3 main components:

* The Scanner - this is a dotnet worker application that runs in the background. It will, every hour, get the latest pricing data for all the stocks we want to monitor (currently just under 90 stocks), it will then combine that current data with historic data for each of them and evaluate each of them based on technical analysis indicators. If we have don't already have a position open we may buy some amount of the stock, but if we do already have a position open, we may sell that position to get out again, and hopefully make some profit in the process.
* The API - this is a dotnet web api application that runs separately from the scanner. Here we can access information about the stocks, their existing price data and the evaluation expressions we use to allow the scanner to allow what to do. We can also see which positions are open and how they are performing.
* The dashboard - this is a react and typescript website we created to monitor the system. Here we can see all the data for each stock, the positions and the overall performance of a single scan (which is how we combine a group of stocks with the evaluation expressions). We can also get real time updates from the scanner here

### How do we decide what stocks to buy and when?

We've handpicked around 90 stocks from the FTSE 350 to start with, but we aim to have the system fully operational on all 350 stocks in future. Being able to monitor so many stocks at once means we are less likely to miss potentially profitable price movements!

We use technical analysis indicators to determine when might be a good time to buy (or sell) any given stock in our system. Before the system can run, we must define some expressions that trigger a buy or sell, we define them using JavaScript and a few custom functions, they can look something like this:

```JavaScript
sma(45) > sma(90) && price(1) > price(12) && sma(5) > sma(8) && sma(3) > sma(4) && sma(2)
```

Here we are using two very simple indicators, the `sma` indicator and the `price` indicator. `sma` stands for Simple Moving Average and is simply an average of some amount of historic data points (this is what the parameter for each function refers to). The other indicator is simply a single price. The parameter for this indicator allows us to reference an exact price, where `price(1)` refers to the most recent price, and `price(4)` refers to the price of 4 hours ago.

Once the system is running, we use a number of different expressions to which each get evaluated against every stock. When the expression returns true, we can either buy, to go long, or sell, to go short on the stock. Once we are in a position for a stock, we use a different set of expressions to decide when the best time to get out is. 

All of our indicators currently require historic pricing data to function properly so it's very very important we get reliable and up to date data when running!

### How do we get historic data?

The backbone of our system lies in the retrieval and storage of historic data. We source this from [IG](https://labs.ig.com/), a broker, using their API, and then store it in our own database. Our system knows exactly which historic data points it needs in order to function and so we can ask for multiple different bits of data, from wildly different dates and times all in an efficient manner. IG has limits on how much data we can retrieve within a week though, so we have to ensure that we don't exceed those limits prematurely.

Once we have the data we need, the custom built JavaScript expression evaluator tells us if we should buy or sell a stock.

### How do we actually buy and sell stocks?

Once we've identified a position to open or close. We connect to IBKR, another broker, to facilitate the trade. This requires us to create and maintain a bit of IBKR software called the IB gateway, which we host for ourselves in AWS
