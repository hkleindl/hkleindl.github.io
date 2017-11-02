---
layout: post
title:      "CLI Data Gem"
date:       2017-11-02 21:52:16 +0000
permalink:  cli_data_gem
---


  I have been looking forward to working on a project from scratch as a way to put everything I've learned so far to the test. The CLI Data Gem has been a great way to do just that. It was a little intimidating finding a place to start, but using Bundler to set up all the basic file structures for me was a huge help. I felt a lot better once I had a lib folder I could start putting some code into. 

  I wanted my gem to scrape a website with upcoming concerts, print a list of the artists, then have the user pick a concert from that list to see more details about the selected concert. Since I had a general idea of how I wanted my interface to work, writing the basic CLI functionality with some placeholder data went pretty quickly. 

  I was feeling pretty good about how things had been going up to that point. Scraping data from the website I chose offered more of a challenge. I used Nokogiri to successfully parse some of the HTML, but I found myself spending way too much time trying to isolate the remaining data. I decided it would be quicker to use a different site with similar information that had easier HTML to scrape. I was able to scrape some of the elements I needed, but out of nowhere, my code stopped working. After driving myself crazy trying to figure out what I had screwed up, I finally realized my code was fine. It was the site that kept breaking my program. I was unaware that some sites don't like to be repetitively scraped. So, yet again, I started over with a third site which was much more cooperative. 

  Once I had successfully parsed each concert's artist, venue, date, and URL, I was ready to fill out my `Concert` class. I defined a few methods in `Concert` but wasn't really feeling great about the direction I was going. I rewatched the video in which Avi discussed common 'anti-patterns' and realized I was setting myself to make similar mistakes. I rethought my approach and decided it would be better to add a designated `Scraper` class that instantiates an instance of a `Concert` object and sets its `attr_accessor :artist, :venue, :date`, and `:url` to the corresponding scraped data. Then, `Scraper` calls the `save` method I defined in `Concert` to keep track of that instance. 

  After I had gotten `Scraper` and `Concert` working together, I replaced the placeholder data in the `CLI` class with the objects created by `Scraper`. I had to refactor a bit of my `CLI` class, but all the heavy lifting had been done. My gem was now able to print a list of artists with upcoming concerts, ask for input to see details about a chosen concert, display the venue and date, and display the URL to buy tickets for that concert. 

  The gem project has presented me with a lot of unique challenges and pushed my knowledge of everything I have learned so far. There were quite a few ups and downs, but I'm pretty happy with how everything turned out. 


