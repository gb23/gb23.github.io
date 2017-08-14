---
layout: post
title:  "Sinatra Refrigerator Inventory app"
date:   2017-08-14 17:19:16 +0000
---



The refrigerator ("Fridge") inventory app provides a database and web interface for users to:
* Create an account which manages user's Fridges and Items contained within.
    * Each user can **ONLY** modify content of his/her own fridge (and its contents within).  The **EXCEPTION** is with shared accounts.  A fridge rarely belongs to just one person, so multiple people can "co-own" a fridge with this app's built-in account ```linking``` feature.
    * A user can create, read, update, and delete (CRUD) any items in a fridge to which he or she is an owner or co-owner of said fridge.
    * A user is also able to perform CRUD operators on any fridges themselves to which he/she owns/co-owns.
    * A user, in addition to sharing fridges with other users, can also view the accounts to which he/she is currently sharing and use an ```unlinking``` feature to unlink any user that is currently linked. 
    * A user is also able to update his/her own information, as well as update current password, and delete his/her account.
    * User inputs are validated for users, fridges, and items creation.

This app was built with Sinatra, extended with [Rake tasks](https://github.com/ruby/rake) for working with an SQL database using [ActiveRecord ORM](https://github.com/rails/rails/tree/master/activerecord).

## Usage

After checking out the repo, run ```bundle install``` to install Ruby gem dependencies.

You can start one of Rack's supported servers using the [shotgun](https://github.com/rtomayko/shotgun) command ```shotgun``` and navigate to `localhost:9393` in your browser.


## Models
The Fridge Inventory app database includes four model classes: ```User, UserFridge, Fridge, and Item```

1. User: 
```
  has_many :user_fridges
  has_many :fridges, through: :user_fridges
  has_many :items, through: :fridges
```
    Has attributes:
    * First Name
    * Last
    * Username
    * Email
    * Password (Secured with [Bcrypt](https://github.com/codahale/bcrypt-ruby) hashing algorithm)

2. UserFridge (join table):
```
  belongs_to :user
  belongs_to :fridge
```
    Has attributes:
    * user_id
    * fridge_id

3. Fridge 
```
    has_many :items
    has_many :user_fridges
    has_many :users, through: :user_fridges
```
    Has attributes:
    * name
4. Item   
```
    has_many :items
    has_many :user_fridges
    has_many :users, through: :user_fridges
```
    Has attributes:
    * name
    * quantity
    * category
    * note
    * Sell by date
    * Expiration date
    * amount (grams)
    * fridge_id
  


