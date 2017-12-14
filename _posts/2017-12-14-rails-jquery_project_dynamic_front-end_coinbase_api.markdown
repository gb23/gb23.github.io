---
layout: post
title:      "Rails-Jquery project: Dynamic front-end, coinbase API"
date:       2017-12-14 19:18:29 +0000
permalink:  rails-jquery_project_dynamic_front-end_coinbase_api
---

Time flies when you are working with javascript and jquery.  Mornings quickly turned to afternoons, and afternoons quickly turned to nights when I worked on this guy (or gal... too soon too tell).  

I will use this blog entry for going over how to get trading data from the [Coinbase API](https://developers.coinbase.com/api/v2?shell#introduction/).   I used the [faraday gem](https://github.com/lostisland/faraday) to make requests to the API.  Prior to receiving the coin trading data from the API url, I was using Coinbase's own Ruby gem.  However, like many things made of Ruby today, it kind of fell out of Coinbase's focus and wasn't well maintained.   I was getting many RateLimitErrors, meaning I was requesting too many prices too fast, which is strange considering the size of my app compared to some monster financial tech company's that probably request similar data.  There must be a better way...  So this is what made me ditch the gem and switch over to faraday and accessing the API with faraday through a URL.  

Reading the Coinbase API documentation, one can see there are many end points, from Wallet endpoints (buys, sells, deposits, withdrawals, etc.) to Merchant endpoints (merchanges, orders, checkouts) to Data endpoints (currencies, exchange rates, prices, and time).  In the latter endpoint, I needed the price.  More specifically, I needed something called the [spot price](https://en.wikipedia.org/wiki/Spot_contract#Spot_prices_and_future_price_expectations).  Whereas the buy price is the price to buy bitcoin, ether, or litecoin, and the sell price is the price to sell a coin, the spot price is the market price for the coin and is most often found between the buy and sell prices.  For my app, I am most concerned with 'given the current market price for a coin, how are my investments doing?'.  The API allows you to make a GET request to `https://api.coinbase.com/v2/prices/:currency_pair/spot` where the `:currency_pair` would be the symbol for the crypto you want followed by a hyphen which is then followed by the fiat currency symbol for conversion.  So for instance, if I requested the spot price of bitcoin in US dollars, `:currency_pair = BTC-USD` where BTC is the symbol for bitcoin and USD is the symbol for US dollars.

There is an optional parameters that you can send with the get request:  the date parameter.  Say you wanted the spot price of bitcoin on October 11, 2012 (so you can cry yourself to sleep for not having bought the coins then) you would send the parameter as a string "YYYY-MM-DD" in UTC and API response would give you your price as a JSON.

Though I did not need to use this parameter (because my app is only interested in current market pricing), with the faraday gem, it would be simple enough:<br>
<code>resp = Faraday.get 'https://api.coinbase.com/v2/prices/ETH-USD/spot'  do |req| 
<pre>     req.params['date'] = "2012-10-11"</pre>
end</code>

Making the request for the current market price, as I did for my add, returns the following data to resp.body:<br>
`"{\"data\":{\"base\":\"BTC\",\"currency\":\"USD\",\"amount\":\"16838.35\"}}".`
I need to get this string into JSON so I can get the amount data I want. (When you are reading this, is the price for bitcoin of $16838.35 now a steal?  I hope so.) If we
`JSON.parse(resp.body)`, the response looks much better:<br>
`{"data"=>{"base"=>"BTC", "currency"=>"USD", "amount"=>"16838.35"}}`.  Getting the amount as a string is just `JSON.parse(resp.body)["data"]["amount"]`.  APIs are just so neat -- so much data everywhere to request and make your own!





