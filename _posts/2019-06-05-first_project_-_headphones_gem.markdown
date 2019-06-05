---
layout: post
title:      "First Project - Headphones Gem"
date:       2019-06-05 06:37:12 -0400
permalink:  first_project_-_headphones_gem
---


My first project's finally in a place where I can feel comfortable writing a blog post about it. My code is probably going to need some refactoring, but for now it feels great knowing that the toughest bit is over (hopefully). So let's start with a summary of what my gem does.

## Overview 
This project was inspired by one of the recent dilemmas I had - finding the perfect pair of wireless headphones that would make me feel like I was paying my money's worth. With so many good, comparable products out there, it can be quite a conundrum for some of us to make that choice, not to mention time-consuming having to click through numerous links to conduct your research. What my app aims to do is simplify the initial process of filtering through multiple headphones based on criteria such as price, design, features, sound quality, and value for money, and subsequently allows the user to view: 

1. a list of retailers that sell a particular model of interest and 
2. a summarized review of pros and cons associated with that product 

My hope for this project is that it quickens the user's research process, ultimately allowing us to spend less time racking our brains and waiting for web pages to load, and spend more time on the more important things in life. 

## User Experience
### Introduction
![Introduction](https://i.imgur.com/OPGQ3D6.png)

The CLI starts by displaying the list of headphones in the sequence it appears on cnet.com and subsequently gives the user 6 options, each representing a category to sort the headphones by. The programme is created such that the user can continue sorting by different categories until he/she enters "exit" to proceed to the next phase of the programme.

![Sort by Design](https://i.imgur.com/p1yFiom.png)

### Picking Specific Headphone
![Picking Specific Headphone](https://i.imgur.com/c0nTpQ0.png)

Once "exit" is entered, the CLI then prompts the user to enter his/her headphone of interest to obtain further details. The number to select will be based on the most recently generated sorted list. 

Once a specific product is chosen, the user can then select to dive deeper into either a price comparison or a summarized review of that product.

### Further Details
Below are some screenshots on what is printed for the price comparison and summarized review respectively.  

![Price Comparison](https://i.imgur.com/slabZRS.png) 

![Summarized Review](https://i.imgur.com/kaupoBd.png)


### Option to Continue Browsing
![Option to Continue Browsing](https://i.imgur.com/AvSzAz2.png)

Lastly, before the CLI programme is terminated, it prompts the user whether they would like to continue browsing. If "y" is entered, the user can choose which point within the programme to return to - either to go back to the initial sorting stage or to backtrack to the stage offering the user to view more details on the most recent list of headphones. 

This particular prompt will keep appearing until the user enters any other key than "y", in which case the programme will simply end with a goodbye message. 


## Behind the Scenes (Coding Process)
Let's start with the #call function within the CLI controller class. 

```
  def call
    puts ""
    puts "Top 15 Headphones to Own for 2019".colorize(:cyan).underline
    make_headphones
    list_headphones
    menu
    more_details
    last_request
    goodbye
  end
```
	
This contains all the methods associated with my programme. For this section, I will just go through select aspects of my code such as my scraping process and how I have employed the use of Object Oriented Programming (OOP) to make up the building blocks of this app. 

To carry this out, I have created three different class, namely: 
1. Headphones :: Scraper
2. Headphone
3. DetailsScraper

### `Headphones :: Scraper`

My code within this class is as follows:

```
class Headphones::Scraper

  def get_page
    Nokogiri::HTML(open("https://www.cnet.com/topics/headphones/best-headphones/stereo-bluetooth/"))
  end

  def stats
    headphones={}
    get_page.css("div.bestMeta a h5").each{|i| headphones[i.text.to_sym]={}} 
    price_array=[]
    get_page.css("div.pricing span.price").each do |i|
      price_array << i.text.gsub("$","").to_i
    end
    stats=[]
    get_page.css("div.subRatings ul").each do |stat| #creating an array of stats (ex-price) for each headphone
      stats_hash={
        design: stat.css(":nth-child(1n) span.rating").first.text.to_i,
        features: stat.css(":nth-child(2n) span.rating").first.text.to_i,
        sound: stat.css(":nth-child(3n) span.rating").text.to_i,
        value:stat.css(":nth-child(4n) span.rating").text.to_i
      }
      stats << stats_hash
    end
    review_links=get_page.css("div.bestMeta a.review").map{|i| "https://www.cnet.com#{i.attr("href")}"}
    comparison_links=get_page.css("div.pricing a.allPrice").map{|j| "https://www.cnet.com#{j.attr("href")}"}
    index=0
    headphones.each do |headphone,stat| 
      headphones[headphone]=stats[index]
      headphones[headphone][:total]=stats[index].values.sum
      headphones[headphone][:price]=price_array[index]
      headphones[headphone][:review_url]=review_links[index]
      headphones[headphone][:comparison_url]=comparison_links[index]
      index+=1
    end
    headphones
  end

end
```

Over here I have created two instance methods: #get_page and #stats. #get_page performs the simple function of returning the html of the webpage where we're going to scrape the core data from - [https://www.cnet.com/topics/headphones/best-headphones/stereo-bluetooth/](https://www.cnet.com/topics/headphones/best-headphones/stereo-bluetooth/). 

#stats performs the more complicated function of scraping the html data to get the data that we require and compile it into a hash (named `headphones`) containing the headphone names as the keys and the attributes such as price, design, features, sound and value as nested hashes with their corresponding values. 

Below is a small section of the `headphones` hash: 

```
=> {:"Sony WH-1000XM3"=>
  {:design=>10,
   :features=>9,
   :sound=>10,
   :value=>8,
   :total=>37,
   :price=>298,
   :review_url=>"https://www.cnet.com/reviews/sony-wh-1000xm3-review/",
   :comparison_url=>"https://www.cnet.com/products/sony-wh-1000xm3/prices/"},
 :"Jabra Elite Active 65t"=>
  {:design=>9,
   :features=>9,
   :sound=>8,
   :value=>8,
   :total=>34,
   :price=>190,
   :review_url=>"https://www.cnet.com/reviews/jabra-elite-active-65t-review/",
   :comparison_url=>"https://www.cnet.com/products/jabra-elite-active-65t/prices/"},
 :"Apple AirPods 2019"=>
  {:design=>9,
   :features=>9,
   :sound=>7,
   :value=>8,
   :total=>33,
   :price=>140,
   :review_url=>"https://www.cnet.com/reviews/apple-airpods-2019-review/",
   :comparison_url=>"https://www.cnet.com/products/apple-airpods-2019/prices/"},
```

I'd like to also note at this point that I have added two symbols - `:review_url` and `:comparison_url` to each headphone's hash. We will be making use of these in the `DetailsScraper` class. 

### `Headphone`
Now that we know what the headphones hash looks like, we can move on to the `Headphone` class where we see the code associated with instantiating headphone objects with their corresponding attributes. 

```
class Headphone

  attr_accessor :name, :price, :design, :features, :sound, :value, :total, :review_url, :comparison_url

  @@all=[]

  def initialize(name,stats)
    @name=name
    stats.each{|k,v| self.send("#{k}=",v)}
    @@all << self
  end

  def self.create_from_collection(collection_hash)
    collection_hash.each{|name,stats| self.new(name,stats)} #makes a new instance out of each headphone hash
  end

  def self.all
    @@all
  end

end
```

In the #initialize method, I have utilized the #send method which is a form of metaprogramming to iterate over each headphone attribute and assign those attributes to the newly instantiated object. The newly instantiated object is then pushed to the @@all class variable, which we use in the CLI controller. 

#self.create_from_collection method is a class method designed to take the headphones hash from the `Headphones :: Scraper` class as an argument, iterate over all headphones and instantiate them using the initialize method (i.e. self.new). 

This is all then culminated within the #make_headphones method of the CLI controller class. 

```
  def make_headphones
    collection_hash=Headphones::Scraper.new.stats
    Headphone.create_from_collection(collection_hash)
    @list=Headphone.all
  end
```

In the first line of code, I have created the collection_hash variable which is basically what the hash that the #stats method returns within the Headphones :: Scraper class. 

I then make use of the Headphone class method to instantiate a Headphone class object for each headphone, complete with all their relevant attributes. 

Lastly, I assign the complete list of Headphone objects to the instance variable @list within our CLI controller class. In this case, I have created @list because my app involves sorting features and @list will change according to the criteria the user decides to sort by, which will then inform how the rest of the programme works. Therefore, it is necessary for the app to keep track of the changes happening to @list in order to return the intended output. 

### `DetailsScraper`
The DetailsScraper class is required for us to scrape data depending on which headphone the user decides to view more details. Thankfully, the data required for us to provide the price comparison or the summarized review exist on the same webpage, so only one round of scraping is needed for the #scrape_prices and #scrape_review methods to work. 

```
class DetailsScraper #creating this class to accommodate the #more_details method in our controller

  attr_accessor :url,:details

  def initialize(url)
    @details=Nokogiri::HTML(open(url))
  end

  def scrape_prices
    sellers=[]
    @details.css("div[section='wtbSmall'] div.col-3").each do |i|
      hash={
        price: i.css("span[section*='price']").text.strip,
        seller: i.css("span[section*='seller']").text.strip
      }
      sellers << hash
    end
    sellers
  end

  def scrape_review
    hash={
      good: @details.css("p.theGood span.content").text.strip,
      bad: @details.css("p.theBad span.content").text.strip,
      bottom_line: @details.css("p.theBottomLine span.content").text.strip,
    }
    hash
  end


end
```

So how does this integrate with our CLI Controller? Starting from the top: 
1. User sorts list of headphones based on desired criteria (e.g. price, value, sound, etc.)
2. User picks specific headphone from the sorted list 
3. The programme then identifies that specific headphone and gets the :review_url associated with that headphone object 
4. The :review_url is then used to return the html and assign it to a local variable @details within the CLI Controller as a new instance of the DetailsScraper class
5. Once we have the html data, the user then decides to view either the price comparison or summarized review and the corresponding #scrape_prices or #scrape_review methods will be run accordingly

### Conclusion
Given this was my first project, I'm absolutely conscious that there's loads of room for improvement, so if anyone's still reading this, please let me know if you have any feedback on how I can better my code or if you have any ideas on interesting features to add. You can find the rest of my code on my [github repository](https://github.com/bo1623/first-project-headphones). 

Thanks for reading!! 

