---
layout: post
title:      "A Recap of Object Oriented Programming"
date:       2019-05-30 03:05:35 +0000
permalink:  a_recap_of_object_oriented_programming
---

### Object Oriented Programming

As Avi rightly mentioned in one of his videos, OOP is a complex topic and requires lots of iteration and practice for the concepts to truly take hold in our minds. Moreover, with the immense breadth of this topic, I thought it would be useful to compile a summary of the relatively tougher concepts and interesting learning points we picked up in this module.

### Object Oriented Programming vs Procedural Programming
“Object-oriented programming is a programming paradigm that uses abstraction (in the form of classes and objects) to create models based on the real world environment. An object-oriented application uses a collection of objects, which communicate by passing messages to request services. Objects are capable of passing messages, receiving messages and processing data. The aim of object-oriented programming is to ***try to increase the flexibility and maintainability of programs***. Because programs created using an OO language are modular, they can be easier to develop, and [simpler to understand after development](http://www.ctp.bilkent.edu.tr/~russell/java/LectureNotes/1_OOConcepts.htm)”  

This pretty much echoes what we’ve been taught so far on OOP. One memorable quote from Avi was that an object is the combination of data and logic /procedures whereas in procedural programming, these are separate. In procedural programming, each time we define a method, an argument needs to be specified in order to take in the data we’re operating on. This can be troublesome as we would have to manage our data through proximity and arguments, and remember the appropriate data structures to be passed into those methods. However, in OOP, methods are defined within the scope of each class, we’re essentially teaching our objects to manage their own data. 



### Object Relationships
#### [The “Belongs To” Relationship ](https://learn.co/tracks/online-software-engineering-structured/object-oriented-ruby/object-relationships/intro-to-object-relationships)

In the lab above, we’ve used a Song class to illustrate the “belongs to” concept. What the lesson says is that songs (or song instances) can have multiple attributes, i.e. belong to these attributes (:title, :artist). In turn, many other song instances could belong to this same artist. So how do we model this dynamic? 

To do this we need to first create two different classes – **Song** and **Artist**

```
class Song

  attr_accessor :title, :artist
	
	def initialize(title)
	  @title=title
	end
	
end
```

```
class Artist

  attr_accessor :name, :genre
	
	def initialize(name, genre)
	  @name=name
		@genre=genre
	end
	
end
```
 
Once this is done, we can simply create an Artist instance and a Song instance:

```
drake = Artist.new("Drake", "rap")
hotline_bling = Song.new("Hotline Bling")
```

We then assign the artist instance to hotline_bling’s instance variable of @artist with the following:

`  hotline_bling.artist = drake`


As a result, the song now belongs to the artist instance of “drake”, and in turn “drake” has other attributes that we can call, for example: 

```
hotline_bling.artist.genre => “rap”
hotline_bling.artist.name => “Drake”
```



#### [The "Has Many Relationship](https://learn.co/tracks/online-software-engineering-structured/object-oriented-ruby/object-relationships/has-many-object)

In the “belongs to” example above, each song instance belongs to an artist. The “has many” relationship is the inverse of this, i.e. each artist has many songs. Essentially what we want to achieve is this: 

    jay_z.songs 

to return an array of jay_z’s songs where jay_z is the instance of the Artist class. 

To do this, we have to build our code in the Artist class: 

```
class Artist
  attr_accessor :name
 
  def initialize(name)
    @name = name
    @songs = []
  end
 
  def add_song_by_name(name, genre)
    song = Song.new(name, genre)
    @songs << song
    song.artist = self
  end
 
  def songs
    @songs
  end
end
```

Over here we’re doing a few things:
1. 
```
@songs = []
```

 We have initialized the Song instance with an empty array represented by the instance variable of @songs.

2. 
```
song = Song.new(name, genre)
@songs << song
```
In #add_song_by_name method, we have created a new instance of the Song class called “song” which is initialized with a @name and @genre. We then add this new Song instance to the array of @songs.

3. 
```
song.artist = self
```
We then assign the song’s @artist to our new instance of Artist (“self”) that we are creating in this code.    

4. The result is that we will have an array (@songs) which consists of the song instances that we have created through the #add_song_by_name method, all of which have “self” assigned as its artist. In other words, our new Artist instance now has many songs and genres associated with it.



#### [Class Inheritance ](https://learn.co/tracks/online-software-engineering-structured/object-oriented-ruby/object-architecture/intro-to-inheritance)
```
class Child < Parent
	
     def method
	     #code to overwrite parent’s method
     end
	
end
```

The above code will cause the Child class to inherit all methods and attributes from the Parent (a.k.a. super) class. The option to overwrite the Parent’s methods is also available if we rewrite the parent method in the Child class with a different function. 



#### [Using ‘super’ to supercharge inheritance](https://learn.co/tracks/online-software-engineering-structured/object-oriented-ruby/object-architecture/super)

```
class Student < User
  def log_in
     super
     @in_class = true
  end
end
```
Using ‘super’ allows us to inherit all functionality in the parent’s #log_in method, eliminating the need for us to re-type the whole method if we wanted to add some more functionality into our #log_in method for the Student class.



#### [Keyword Argument ](https://learn.co/tracks/online-software-engineering-structured/object-oriented-ruby/metaprogramming/mass-assignment)

Before Keyword Argument: 
```
def happy_birthday(name, current_age)
   puts "Happy Birthday, #{name}"
   current_age += 1
   puts "You are now #{current_age} years old"
end
```

After Keyword Argument: 

```
def happy_birthday(name: "Beyonce", current_age: 31)
  puts "Happy Birthday, #{name}"
  current_age += 1
  puts "You are now #{current_age} years old"
end
```

Keyword arguments eliminate the need for us to remember the sequence that arguments have to be passed into a certain method. They allow us to name the arguments that we pass in as keys in a hash. Then, the method body uses the values of those keys, referenced by name. So even if we were to change the order of our key/value pairs, the method would still work as intended. E.g. the code below would still output the same result as the above: 

```
def happy_birthday(current_age: 31, name: "Beyonce")
  puts "Happy Birthday, #{name}"
  current_age += 1
  puts "You are now #{current_age} years old"
end
```


### Interesting Learning Points - Infinite Loop 
So this was a problem I encountered while attempting the [Music Library CLI Lab](https://learn.co/tracks/online-software-engineering-structured/object-oriented-ruby/final-projects/music-library-cli) When I tried running the 004_songs_and_artists_spec.rb test file, it resulted in an infinite loop which looked like this: 

```
     Failure/Error: song.artist=self if song.artist==nil
     
     SystemStackError:
       stack level too deep
     # ./lib/artist.rb:36:in `add_song'
     # ./lib/song.rb:38:in `artist='
     # ./lib/artist.rb:36:in `add_song'
     # ./lib/song.rb:38:in `artist='
     # ./lib/artist.rb:36:in `add_song'
     # ./lib/song.rb:38:in `artist='
     # ./lib/artist.rb:36:in `add_song'
     # ./lib/song.rb:38:in `artist='
     # ./lib/artist.rb:36:in `add_song'
     # ./lib/song.rb:38:in `artist='
     # ./lib/artist.rb:36:in `add_song'
     # ./lib/song.rb:38:in `artist='
     # ./lib/artist.rb:36:in `add_song'
     # ./lib/song.rb:38:in `artist='
     # ./lib/artist.rb:36:in `add_song'
```

This was arising from my setter method #artist= in my Song class, where I referenced the #add_song method which lies within the Artist class. 

```
  def artist=(artist)
    artist.add_song(self) unless artist.songs.include?(self)
  end
```

The solution to this infinite loop is to add “@artist= artist” within the #artist= setter method, which aims to assign the argument to the instance variable of @artist within the Song class. So the correct solution should be: 

```
  def artist=(artist)
    @artist=artist #this is important to prevent infinite loop
    artist.add_song(self) unless artist.songs.include?(self)
  end
```


Why is that? What I’m going to do here is compare the first scenario with the second one. But first, here is what the #add_song method looks like within the Artist class.  

```
  def add_song(song)
    song.artist=self if song.artist==nil
    @songs << song unless songs.include?(song)
  end
```

What this method does is take in a Song instance as the argument and assign its @artist to be an instance of itself if the song does not have an artist already, hence the reason for the “if” condition. Now notice that “song.artist=self” is actually the setter method within the Song class, so this would bring us back to the #artist= setter method within the Song class, which then brings us back to the #add_song within the Artist class, creating an infinite loop. 

This is why “@artist=artist” is important because once the artist instance is assigned to @artist within the Song class, when the #add_song method is called, the setter method of “song.artist=self” will not be called because song.artist is no longer ‘nil’. This then prevents an infinite loop from happening.

### Methods to Know: 
#### The `send` method
This is a form of metaprogramming which comes in handy when scraping data from websites that are ever-changing. 

Basically: 

`object.send(key=, value)`

is the same as

`object.key = value`

As illustrated in the [Twitter Example](https://learn.co/tracks/online-software-engineering-structured/object-oriented-ruby/metaprogramming/mass-assignment-and-metaprogramming), this comes in handy when the objects we’re trying to scrape don’t stay constant and we don’t want to keep changing our code to accommodate the changing attributes that we’re trying to scrape. In essence, we’re trying to abstract away our class’ dependency on specific attributes. This metaprogramming method enables us to tell our class to accept some unspecified number and type of attributes and assign those accordingly within our class.

```
class User
  attr_accessor :name, :user_name, :age, :location, :bio
 
  def initialize(attributes)
    attributes.each {|key, value| self.send(("#{key}="), value)}
  end
end
```

#### The `tap` method

```
def create_by_name(name)
   self.new.tap do |i|
i.name = name
   end
end
```

is the same as 

```
def create_by_name(name)
   i=self.new
   i.name=name
   i
end
```

So what #Tap does is assign the variable within the pipes as the object, carries out the the functionality within the block and then returns the object at the end of the block. 


