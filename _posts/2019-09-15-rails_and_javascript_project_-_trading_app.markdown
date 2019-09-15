---
layout: post
title:      "Rails & Javascript Project  - Trading App"
date:       2019-09-15 13:26:46 -0400
permalink:  rails_and_javascript_project_-_trading_app
---


For my Rails & Javascript Project, I decided to create something relevant to my field of work - finance - and came up with the simplest of trading apps (although the process of coding it was far from simple). Before I delve into the code, here is a preview of what my app looks like: 

![pic1](https://i.imgur.com/XrZWSBq.png)

![pic2](https://i.imgur.com/vuvTsk3.png)

Gotta say, I'm decently proud of what I've produced here and I hope it's not just personal bias arising from the many hours spent on creating this app. 

A short introduction - in the first screenshot we have a 5-month price chart of Medtronic (ticker: MDT) that was rendered to the DOM by entering its ticker in the search bar on top. To the right of that search bar is the current user's details, together with a logout button. Right below the chart are four buttons - the first two enable the user to toggle between viewing the stock's 5-month price chart and its intraday price chart, and last two buttons on the right allow the user to buy or sell the stock whose chart is being rendered on the DOM. 

The second screenshot portrays the user's cash balance along with a "deposit" and "withdraw" button, which allows the user to transfer USD cash in and out of the trading account. Further down the page, we have the portfolio positions associated with the user's account in table form and a pie chart depicting the user's full portfolio composition by weight. 

Lastly, on the right of the page is a sidebar with the day's top news, where the user can search for news by country or by keyword, just like having a mini Google window on the side of your page.

### Credit Where Credit's Due
Besides creating my own APIs using Rails and using those APIs to render information onto the DOM, putting this all together required the use of some third party tools, to which some acknowledgement is due. To obtain the stock price data, I made use of [Alpha Vantage](https://www.alphavantage.co/) APIs, a free API provider for US-listed stocks. And for the news sidebar, I used APIs offered by [newsapi.org](https://newsapi.org/), a handy, user-friendly tool for getting worldwide news with customizable search crtieria. Lastly, the plotting of my stock price chart and portfolio pie chart were powered by [plotly.js](https://plot.ly/javascript/). If you're looking to incorporate similar features in your future projects/apps, I'd recommend using the providers above (if you're particularly tight on budget). 

### Breaking It All Down
As usual, I will start by explaining the model relationships involved in this app. The four models are: 

1. User
2. Stock
3. Trade
4. Position

And their corresponding relationships are as follows: 

```
class User < ApplicationRecord
  has_many :trades
  has_many :positions
end
```

```
class Stock < ApplicationRecord
  has_many :positions
  has_many :trades
end
```

```
class Trade < ApplicationRecord
  belongs_to :stock
  belongs_to :user
end
```

```
class Position < ApplicationRecord
  belongs_to :stock
  belongs_to :user
end
```

A User simply represents a particular account with a particular portfolio of stocks and cash. So within a particular User's account, that User would be able to make Trades via the app, which in turn would affect his/her stock Positions. If the user bought a certain number of Apple shares, his/her position in that stock would increase by that amount, vice versa if the User was selling them. 

Whenever a User executes a Trade in a Stock that does not yet exist in the backend, a new Stock instance would be created with the relevant stock ticker. It would be natural to assume that this particular Stock would be traded many times by Users in different Trades, and be held in many Users' portfolios, hence its has_many trades and positions relationships. You may have noticed that a couple of "has_many through" exist here given a User has many Positions and has many Trades, however I did not model these as I could not see a clear use case for them. 

A Trade instance is created each time a user executes a buy or sell trade, and naturally it belongs to a stock. 

Lastly, each time a User purchases a Stock that isn't in his/her portfolio, a new Position instance is added which belongs to that particular Stock and that User. If the User makes future Trades in Stocks that already exist under his/her account, then that Position gets updated accordingly. 

### Behind The Scenes
Since this project is meant to demonstrate our understanding of the Rails-Javascript interaction, I will focus my post on those aspects. A good example of how I've demonstrated this is how I  translated a user's trade from the frontend and posted that to the backend to be saved as a new Trade instance in Rails, and then use the resulting API to render new information on the DOM. In order to do this, I created a Trade class within Javascript with a standard constructor function and a `postTrade()` instance function to execute what I just described:

```
class Trade{
  constructor(ticker,username,price,direction,quantity){
    this.ticker=ticker
    this.username=username
    this.price=price
    this.direction=direction
    this.quantity=quantity
  }

  postTrade(){
    fetch("http://localhost:3000/trades",{
      method:'POST',
      headers: {
        "Content-Type":"application/json",
        "Accept": "application/json"
      },
      body: JSON.stringify(this) //use .then over here to update table contents and render new trade onto DOM
    })
    .then(resp=>resp.json())
    .then(json=>newTradeUpdateTable(json))
    .then(userjson=>updateCashInDOM(userjson))
    .then(updatePieChart)
  }
}
```

Now that we know what a Trade instance looks like and what it can do, here's the event listener I added to the Trade modal form that enables the user to execute a trade: 

![Trade Form](https://i.imgur.com/MsDrjEK.png)


```
document.getElementsByClassName('modal-content')[0].addEventListener('submit',function(event){
  let ticker=document.querySelector('.modal-content h3').innerText.split(' ')[1]
  let direction=document.querySelector('.modal-content h3').innerText.split(' ')[0]
  let username=document.querySelector('#logged-in-user').innerText.split(' ')[1]
  let tradePrice=document.getElementById('real-time-price').innerText.split(' ')[1]
  let quantity=document.getElementById('number-of-shares').value
  let trade=new Trade(ticker,username,tradePrice,direction,quantity)
  trade.postTrade()
  modal.style.display="none"
  event.preventDefault()
})
```

The first five lines of code are declaring variables which hold the data we want to create our new Trade instance with and the sixth line initiates a new Trade instance in our current browser session, labelled as "trade". I then call the `postTrade()` method on that newly created Trade instance to post that trade to the backend in Rails, which brings us to the TradesController in Rails:

```
class TradesController < ApplicationController

  def create
    user=User.find_by(username: params[:username])
    stock=Stock.find_or_create_by(ticker: params[:ticker])
    trade = Trade.create(stock_id: stock.id, user_id:user.id, price:params[:price], direction:params[:direction],  quantity:params[:quantity])
    position=Position.find_or_create_by(user_id: user.id,stock_id: stock.id)
    position.update_position(price:params[:price].to_f,direction:params[:direction],quantity:params[:quantity].to_i)
    render json: PositionSerializer.new(position).to_serialized_json
  end

end
```

In this create action, we are finding the User instance who just executed that trade, and a stock instance is retrieved or created depending on whether a Stock instance with that ticker exists in the database. A new Trade instance is then created by Rails that belongs to that user and stock with other attributes such as executed price, direction (buy or sell) and quantity (number of shares traded). In this action, the user's position in this stock is also updated (or created) in the database using the `update_position()` instance method for Position instances. 

Lastly and most importantly, after the update is carried out, the Position instance is then rendered as a json via the PositionSerializer class, show below: 

```
class PositionSerializer

  def initialize(position)
    @position=position
  end

  def to_serialized_json
    options={
      include:{
        stock:{
          only: [:ticker,:latest_price]
        },
        user:{
          only: [:username,:cash_balance]
        }
      }
    }
    @position.to_json(options)
  end

end
```

In addition to the Position's attributes such as size, stock_id, user_id, cost, value, realized (profit) and unrealized (profit). The frontend woul also require additional information from the Position's associated Stock and User instances, hence the `options` variable under the `to_serialized_json` method. 

Now if you refer back to the `postTrade()` function within the Trade class block, after the position json Promise is returned, I run three functions: 

```
    .then(json=>newTradeUpdateTable(json))
    .then(userjson=>updateCashInDOM(userjson))
    .then(updatePieChart)
```

The first function simply involves updating the position table within the DOM using the position json that was rendered at the end of the create action within the TradesController. 

```
function newTradeUpdateTable(position){
  let ticker=position.stock.ticker
  if(!!document.getElementById(`${ticker}-size`)){
    let latest_price=document.getElementById(`${ticker}-price`)
    let size=document.getElementById(`${ticker}-size`)
    let cost=document.getElementById(`${ticker}-cost`)
    let value=document.getElementById(`${ticker}-value`)
    let unrealized=document.getElementById(`${ticker}-unrealized-profit`)
    let realized=document.getElementById(`${ticker}-realized`)
    latest_price.innerText=Number(position.value/position.size).toFixed(2)
    size.innerText=position.size
    cost.innerText=Number(position.cost).toFixed(2)
    value.innerText=Number(position.value).toFixed(2)
    unrealized.innerText=Number(position.unrealized).toFixed(2)
    realized.innerText=Number(position.realized).toFixed(2)
  }else{
    let table=document.querySelector('table')
    table.innerHTML+=`
      <tr id=${ticker}-position-details>
        <td>${ticker}</td>
        <td id="${ticker}-price">${Number(position.value/position.size).toFixed(2)}</td>
        <td id="${ticker}-size">${position.size}</td>
        <td id="${ticker}-cost">${Number(position.cost).toFixed(2)}</td>
        <td id="${ticker}-value">${Number(position.value).toFixed(2)}</td>
        <td id="${ticker}-unrealized-profit">${Number(position.unrealized).toFixed(2)}</td>
        <td id="${ticker}-realized">${Number(position.realized).toFixed(2)}</td>
        <td ticker="${ticker}"><button class="update-price-btn">Update</button></td>
      </tr>
    `
  }
  return position.user //to be used in updateCashInDOM()
}
```

Notice how at the end of this function, I have returned the position's user to be used in the next function - `updateCashInDOM`, which takes the user's json containing `cash_balance` as one of its attributes and renders it onto the DOM. 

```
function updateCashInDOM(user_json){
  let div=document.getElementById('cash-balance')
  div.innerText=`Cash Balance: ${Number(user_json.cash_balance).toFixed(2)}`
}
```

Finally, the last function doesn't require an argument but it is important that this function is only called after all information has been updated in the backend and the new cash balance is reflected in the DOM, as the `createPieChart()` function makes use of the newly rendered cash balance in the DOM. 

```
function updatePieChart(){ //fetches json of all existing positions and creates new pie chart with them
  let username=document.querySelector('#logged-in-user').innerText.split(' ')[1]
  fetch("http://localhost:3000/positions",{ //need to post instead of get because we want to post the username to the positions controller
    method: "POST",
    headers: {
      "Accept": "application/json",
      "Content-Type": "application/json"
    },
    body: JSON.stringify({username: username})
  })
  .then(resp=>resp.json())
  .then(json=>createPieChart(json))
}
```

If you're interested, here is what the `createPieChart()` function looks like. It works by taking the array of a particular user's positions, declaring array variables containing each stock's ticker and their corresponding values respectively. The cash balance and the label "Cash" is also added to these arrays before being passed into the Plotly.js plotting function. 

```
function createPieChart(array){ //array of position objects
  let tickers=array.map(pos=>pos.stock.ticker)
  let values=array.map(pos=>Number(pos.value))
  tickers.push('Cash')
  let cashBalance=Number(document.querySelector('#cash-balance').innerText.split(' ')[2])
  values.push(cashBalance)
  console.log(values)
  console.log(tickers)
  let colors=['rgb(255,71,19)',
  'rgb(255,206,0)',
  'rgb(252,155,179)',
  'rgb(0,139,92)',
  'rgb(144,98,188)',
  'rgb(255,255,255)',
  'rgb(192,11,40)']
  let data = [{
    values: values,
    labels: tickers,
    hole: .4,
    type: 'pie',
    marker: {
      colors: colors
    },
  }];

  let layout = {
    height: 400,
    width: 500,
    title:{
      text:"Portfolio Composition",
      font:{
        size: 24,
        color: 'rgb(255,255,255)'
      }
    },
    font:{
      color: 'rgb(255,255,255)'
    },
    plot_bgcolor:"black",
    paper_bgcolor:"#000000"
  };

  Plotly.newPlot('pie-chart', data, layout);

}
```

And there you have it, a not-so brief overview of my Rails-Javascript project. There's so much more functionality I have incorporated into this app that would take thousands more words to explain, so I'll just stop here and if you're interested in learning more about this app, feel free to visit my github [repository](https://github.com/bo1623/Javascript-Project-Stock-Trading-App).

Thanks for reading :)
