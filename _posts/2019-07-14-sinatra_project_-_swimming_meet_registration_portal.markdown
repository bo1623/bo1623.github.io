---
layout: post
title:      "Sinatra Project - Swimming Meet Registration Portal"
date:       2019-07-14 13:12:26 -0400
permalink:  sinatra_project_-_swimming_meet_registration_portal
---


Safe to say, getting my second project up and running was significantly tougher than my first project on the CLI app. In particular, it was the use of multiple controllers, multiple models, interactions with the database, additional elements that web development introduces (e.g. concept of sessions, mechanics of modelling user login/logout functions using bcrypt) that made this project more challenging but also that much more fulfilling. 

Now for a little context on my project - I used to be a competitive swimmer for more than 10 years and attended many competitions during those times, so I thought it would be interesting to model how the organizers efficiently facilitated the registration process. Each competition had many teams, and each team had many swimmers belonging to those teams, and each swimmer signed up for multiple events (e.g. 100m Butterfly, 100m Freestyle, 100m Breaststroke, etc.), each with their own distinct personal best timings. 

This should give you an idea of the classes involved in my project, which to be honest didn't occur immediately to me at first (I guess one of the many challenges programmers face is learning how to categorize real life objects and develop a skill for detecting and modeling these inter-object relationships). For clarity, the relationships I modeled were: 

```
class Events
  has_many :swimmer_events
  has_many :swimmers, through: :swimmer_events
  has_many :timings
end

class SwimmerEvent 
  belongs_to :swimmer
  belongs_to :event
end

class Swimmer 
  belongs_to :team
  has_many :swimmer_events
  has_many :events, through: :swimmer_events
  has_many :timings
end

class Team 
  has_secure_password
  has_many :swimmers
  has_many :swimmer_events
  has_many :events, through: :swimmer_events
end

class Timing 
  belongs_to :swimmer
  belongs_to :event
end

```

As part of my web application, a new team can create a new account via the sign up function, all they need is a username (team name in this case) and a password. They can then log in, and register new swimmers by keying in the required details such as name, gender and age. 

![Swimmer Sign Up](https://i.imgur.com/aEEom6e.png)


Once this is done, the new swimmer created will appear under the team's list of swimmers, each having a link that directs the user to each individual swimmer's details page, where the user can edit the swimmer's details or sign them up for events. 

![Swimmer List](https://i.imgur.com/6UYWCxV.png)

![Swimmer Profile Page](https://i.imgur.com/HJUmzBw.png)

The event sign up page is flexible in that it allows the user to select a particular swimmer that has been registered from a drop down list. The user then has to tick the checkboxes below to select which events they would like to sign the swimmer up for. 

![Swimmer Race Sign Up](https://i.imgur.com/InyF80d.png)

Upon clicking 'submit', the user is then redirected to another page where we can enter the swimmer's timing for each event. This is where the Timing class comes in. Given each Event object has multiple swimmers signing up for it and that each swimmer has their own distinct timing for that event, I needed to create the Timing class that belongs to a particular swimmer and a particular event. 

![Timing Entry](https://i.imgur.com/gnvpKTn.png)

```

<form method="post" action="/events/register_swimmer/<%=@swimmer.slug%>">

  <label>Please insert swimmer personal best timings below: </label><br>
  <%@swimmer.events.each do |event|%>
    <label><%=event.name%></label><input type="time" name="[swimmer][<%=event.name%>][timing]" min="00:00:00" max="00:30:00" step="1"><br>
  <%end%>

  <input type="submit" value="submit">

</form>
```

The params hash from the form above looked like this: 

```
{"swimmer"=>{"Boys 100m Butterfly - A Division"=>{"timing"=>"00:00:53"}, "Boys 200m Individual Medley - A Division"=>{"timing"=>"00:01:45"}}, "slug"=>"chad-le-clos"}
```

The code below was then used to translate the params hash into creating a Timing object for each entry the swimmer made. 

```
  post '/events/register_swimmer/:slug' do
    @swimmer=Swimmer.find_by_slug(params[:slug])
    @events=@swimmer.events
    signed_up_events=@swimmer.timings.map{|timing| timing.event.name} #control to prevent swimmers from signing up for the same event twice, changing details or race timing can only be carried out via the "edit" function 
    @events.each do |event|
      if signed_up_events.include?(event.name) #if the swimmer has signed up already
        next
      else #if the swimmer hasn't signed up for the event already
        @timing=Timing.create(personal_best: params[:swimmer][event.name][:timing])
        @timing.swimmer=@swimmer
        @timing.event=event
        @timing.save
      end
    end
    redirect "/swimmers/#{@swimmer.slug}"
  end
```

Subsequently, in order to create a new Timing object associated with those particular events and that particular swimmer, I iterated through each event that the swimmer has signed up for and created a new Timing object for each of those events and associated with that swimmer and those specific events using the following lines:

```
@timing.swimmer=@swimmer
@timing.event=event
```

Once the swimmer is signed up for his/her events, the user is redirected to the swimmer's profile page, showing his/her events and their respective personal best timings. Note in particular that each event has a link which directs the user to the event page, showing all the swimmers that have registered for that event, their team names and their respective entry timings. 

![Swimmers in Race](https://i.imgur.com/siEiftp.png)

There are other features that my web app has along the lines of the CRUD framework, you're welcome to check out my code on my Github repository if you're [interested](https://github.com/bo1623/sinatra-project-swimming-competition). As it stands, there are a few ideas that I have to improve on my current app:

1. Incorporate more detail into the swimmer sign up process such as:
* asking for a date of birth, instead of just an age number represented by one integer 
* making relevant events appear only to the swimmer who are eligible (i.e. within the right age group and gender)
2. Making options in my datalists compulsory
3. Create a new feature for generating list of heats for each event 


