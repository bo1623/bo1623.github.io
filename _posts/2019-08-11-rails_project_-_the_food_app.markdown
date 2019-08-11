---
layout: post
title:      "Rails Project - The Food App"
date:       2019-08-11 09:20:35 -0400
permalink:  rails_project_-_the_food_app
---


For my Rails Project, I tried my hand at building a food delivery app called The Food App (resulting from my severe lack of originality). As with each subsequent project, this was significantly tougher than the last and reconfirms my view that the more I learn, the challenge lies increasingly with answering "What's the best way to do this?" instead of "How do I do this?". 

Not planning out your project thoroughly enough before you start coding can have some painful consequences, which I learned the hard way through many hours of refactoring. The key thing to note here is that I *learned*, which is what we're all here to do. 

Anyways, let's get down to it. My focus this post is going to be on a few of the main challenges I encountered while building this app. 

1. Enabling a customer to specify the quantity of dishes to order and assigning those dishes to that customer's meal
2. Allowing OAuth and normal login features to co-exist
3. Assigning ratings for each dish and translating that raw data to average dish ratings and overall restaurant ratings 

For context, I used 7 models for my app, namely User, Restaurant, Dish, Meal, MealsDish, Cuisine and Location. Their relationships are as follows: 

```
class User < ApplicationRecord
  has_secure_password
  belongs_to :restaurant, optional: true
  has_many :meals
  has_many :dishes, through: :meals
  validates :username, presence: true
end

class Restaurant < ApplicationRecord
  belongs_to :user
  has_many :dishes
  has_many :meals, through: :dishes
  belongs_to :location
  belongs_to :cuisine
  accepts_nested_attributes_for :dishes, allow_destroy: true, update_only: true, reject_if: proc{|attributes| attributes[:name].blank?}
end

class Meal < ApplicationRecord
  belongs_to :user
  has_many :meals_dishes
  has_many :dishes, through: :meals_dishes
end

class Dish < ApplicationRecord
  belongs_to :restaurant
  has_many :meals_dishes
  has_many :dishes, through: :meals_dishes
  has_many :users, through: :meals
end

class MealsDish < ApplicationRecord
  belongs_to :meal
  belongs_to :dish
end

class Location < ApplicationRecord
  has_many :restaurants
  has_many :dishes, through: :restaurants
  has_many :meals, through: :dishes
  has_many :users, through: :meals
  validates :name, presence: true
end

class Cuisine < ApplicationRecord
  has_many :restaurants
  has_many :dishes, through: :restaurants
  has_many :meals, through: :restaurants
  validates :name, presence: true
end
```

Lots of models - but what matters for this post are the first five models: User to MealsDish. 

### Enabling a customer to specify the quantity of dishes to order and assigning those dishes to that customer's meal

For context, I envisioned my order form to look like this, where the customer would be able to view all items on a restaurant's menu and then decide how much of a certain dish they wanted before submitting the form.
![Order Form](https://i.imgur.com/i1hR4hK.png)

The desired result was for the customer's meal to be created and saved to our database, the customer would then be able to see their final meal order before proceeding with their purchase and the dishes that had been ordered would also have this reflected in their database.  

![Order Review](https://i.imgur.com/uid2rso.png)

Each dish in a restaurant's menu represents and instance of the Dish class and each customer's order represents an instance of the Meal class. Therefore, it's easy to see that we have a many-to-many relationship here, where a meal has many dishes and a dish has many meals. Therefore, I created a join class called MealsDish that contained a meals_dish_rating attribute alongside meal_id and dish_id, but more on that later. 

We start off in the MealsController where a new Meal instance is created: 

```
  def new
    @restaurant=Restaurant.find(params[:restaurant_id])
    @dishes=@restaurant.dishes
    @meal=Meal.new(user_id: params[:user_id])
  end
```

In my new meal form, we then use a form_for for the new @meal that has been instantiated: 

```
<%= form_for @meal do |f| %>
  <%=f.hidden_field :user_id %>
  <%=hidden_field_tag :restaurant_id, @restaurant.id%>
  <h3>Menu</h3>
  <table>
    <tr>
      <th>Name</th>
      <th>Price</th>
      <th>Quantity</th>
    </tr>
  <%@dishes.each do |dish|%>
    <tr>
      <td><%=dish.name %></td>
      <td><%=dish.price%></td>
      <td><%=number_field_tag "dish_quantities[]",0,min:0,max:10%></td>
      <%= hidden_field_tag "dish_ids[]", dish.id%>
    </tr>
  <%end%>
  </table>
  <%= f.submit "Place your order" %>
<%end%>
```

After hitting submit, what we have is the params hash below: 

```
<ActionController::Parameters {"utf8"=>"✓", "authenticity_token"=>"XclGgb2j5Lxe2frA/UwL8O4bJwEnlySJjJmeXLH87BWqDZIzF8ccrr+Z5tTtSXWDz83BeaT6iG1ZgDMuroBbyg==", "meal"=>{"user_id"=>"8"}, "restaurant_id"=>"6", "dish_quantities"=>["1", "2", "1", "4"], "dish_ids"=>["19", "20", "21", "22"], "commit"=>"Place your order", "controller"=>"meals", "action"=>"create"} permitted: false>
```

When creating a meal, with each dish that is ordered, a corresponding number of MealsDish instances would have to be created, e.g. in our image above, 2 BBQ Pork Buns were ordered, meaning 2 separate instances of MealsDishes needed to be created. To do this, I had to incorporate the :dish_quantities into our create action to create a number of meals_dishes for each specific dish corresponding to the quantity that the customer had ordered. 

```
  def create
    session[:restaurant_id]=params[:restaurant_id]
    if params[:dish_quantities].uniq == ['0'] #condition where all dish_quantites are 0
      redirect_back(fallback_location: root_path) #redirects back to previous page (restaurant show page)
    else
      @meal=Meal.create(meal_params)
      params[:dish_ids].zip(params[:dish_quantities]).each do |id,quantity|
        quantity.to_i.times do #creating quantity times of meals_dish instances
          MealsDish.create(meal_id: @meal.id, dish_id: id) #creating new instances of meals_dishes
        end
      end
      redirect_to user_meal_path(@user, @meal)
    end
  end
```

There you have it, that's how I created a single form to create multiple instances of meals_dishes across multiple dishes. 

### Allowing OAuth and normal login features to co-exist
This was a bit tricky given we only scratched the surface of OAuths in our labs and I haven't even begun to comprehend the various nuances of this gem. My objective was to incorporate a way for a user to login via the normal way (entering a username and password) and via Github (clicking on the link to login via Github, followed by a password specific to the app). 

To my understanding, OAuth logins shouldn't normally require a password but given all user instances in my app require password attributes due to the has_secure_password macro, I had to require users logging in via Github to enter a password on a separate page: 

<blockquote class="imgur-embed-pub" lang="en" data-id="a/MUOGgHy" data-context="false"><a href="https://i.imgur.com/9w2RZNY.mp4">Rails Project - Food App</a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>

To put the normal login and Github login features together, I created two helper methods: 

```
  def omniauth_login
    if User.exists?(uid: session[:omniauth]['uid']) #if we have a user that has logged in via github before
      @user = User.find_by(uid: session[:omniauth]['uid'])
      if @user.authenticate(params[:password])
        session[:user_id] = @user.id
        redirect_to root_path
      else
        flash[:message]="Invalid Password"
        render 'sessions/omniauth' #render if invalid password
      end
    else #if new user logging in via github, we need to create a new user
      @user = User.new(uid: session[:omniauth]['uid']) do |u|
        u.username = session[:omniauth]['info']['nickname']
        u.password = params[:password]
      end
      @user.save
      session[:user_id] = @user.id
      redirect_to root_path
    end
  end

  def normal_login #if the user chooses to login via the normal way
    @user = User.find_by(username: params[:username])
    if !@user.nil? &&  User.exists?(@user.id) && @user.authenticate(params[:password])
      session[:user_id]=@user.id
      if @user.restaurant_manager
        redirect_to restaurant_path(@user.restaurant) and return
      else
        redirect_to root_path and return
      end
    else
      render :new #rendering the log in page again after unsuccessful attempt
    end
  end
```

And implemented it within the create function in our SessionsController as such: 

```
  def create
    if !session[:omniauth].nil? #if the user chose to log in via github
      omniauth_login
    else
      normal_login
    end
  end
```

It is also worth mentioning that I created a 'get' route for my omniauth login in config/routes: 

```
get '/auth/github/callback', to: 'sessions#omniauth'
```

In our SessionsController, this action fulfills the following:

```
  def omniauth
    session[:omniauth]=auth #saving the omniauth data to our session
    render 'sessions/omniauth'
  end
	
	  private

  def auth
    request.env['omniauth.auth']
  end
```

The omniauth action saves request.env['omniauth.auth'] hash to our session to be carried on to our create action. Now, if our session contains the omniauth hash, we will then login via github and request the user to input his/her app password to complete the login. If our session hash does not contain the omniauth hash, the SessionsController just handles that login like any normal login, with the user typing in a username and password to be authenticated. 

### Assigning ratings for each dish and translating that raw data to average dish ratings and overall restaurant ratings 

I mentioned previously that my join class - MealsDish - had an additional attribute called meals_dish rating, this rating is based on what customers rate dishes after having a meal. 

![Ratings Form](https://i.imgur.com/i1hR4hK.png)

This gives rise to the params hash below which gets fed into our update action within our MealsController: 

```
<ActionController::Parameters {"utf8"=>"✓", "_method"=>"patch", "authenticity_token"=>"RSMTzMayJRtafAk25kc/It+B5sbQmDTzxIAMjTZxYgyy58d+bNbdCbs8FSL2QkFR/lcAvlP1mBcRmaH/KQ3V0w==", "dish_ratings"=>["4", "4", "5", "3"], "names"=>["Shrimp Dumplings", "BBQ Pork Buns", "Beef Rice", "Egg Rolls"], "restaurant_id"=>"6", "commit"=>"Submit Ratings", "controller"=>"meals", "action"=>"update", "user_id"=>"7", "id"=>"46"} permitted: false>
```

Within the update action, I go through each element in the dish_ratings and names arrays in tandem with each other using the zip method. Within each iteration, I find all meals_dishes instances with the meal_id of @meal.id and a dish_id corresponding to the dish name and which belongs to the relevant restaurant (recall that if the customer ordered 2 BBQ Pork Buns, there would be 2 meals_dishes instances that were created). For each of these meals_dishes instance, I then assign the rating chosen by the user appropriately. 

```
def update
    params[:names].zip(params[:dish_ratings]).each do |name, rating|
      meals_dishes=MealsDish.where(meal_id: @meal.id, dish_id: Dish.find_by(name: name, restaurant_id: params[:restaurant_id]).id) 
      meals_dishes.each do |meals_dish|
        meals_dish.meal_dish_rating=rating
        meals_dish.save
      end
    end
    @meal.update_dish_rating
    @meal.update_restaurant_rating
    session.delete :restaurant_id #delete the restaurant_id since the cycle of the order has been completed
    redirect_to root_path
  end

```

Subsequently, I make use of two helper methods for meal instances to feed the raw meals_dish_ratings into an overall dish rating or overall restaurant rating: 
1. update_dish_rating
2. update_restaurant_rating 

Helper methods: 

```
  def update_dish_rating
    dishes=self.dishes.uniq
    dishes.each do |dish|
      meals_dishes = MealsDish.where(dish_id: dish.id).where.not(meal_dish_rating: nil)
      meals_dish_ratings= meals_dishes.map{|meals_dish| meals_dish.meal_dish_rating}
      dish.dish_rating=(meals_dish_ratings.sum/meals_dish_ratings.size).to_i
      dish.save
    end
  end

  def update_restaurant_rating
    dishes=self.dishes.uniq
    restaurant=dishes[0].restaurant
    restaurant.weighted_rating(dishes)
  end

```

One additional feature I would like to add to my app is the ability for restaurant managers to track which of their dishes contribute the most to their overall revenue. Given each dish has a price, this should be quite doable. But given the project time constraint, you'll have to settle for this version for now. 

That was a lengthy one, if you're still reading, thank you for staying with me till the end! 








