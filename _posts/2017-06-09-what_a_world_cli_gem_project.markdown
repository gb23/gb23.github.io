---
layout: post
title:  "What A World CLI gem project"
date:   2017-06-09 16:29:27 -0400
---


Check out the gem!: 
[![1.1](https://badge.fury.io/rb/what-a-world.svg)](https://badge.fury.io/rb/what-a-world)

What a World!  Exploring the world is always an adventure.  This gem provides C.I.A. data on transnational issues from various locations arould the planet. Caution: Results may be shocking.  The idea behind this gem is to present the user with troubling issues with the intent of raising awareness of global issues.  Hopefully, you will learn something new! 

[Github Repo](https://github.com/gb23/what-a-world-cli-gem).

The program begins by asking the user to search for a global location by typing the first letter of the location, A through Z.  Then an enumerated list of locations is generated based on that letter.  The user then chooses a location by number.  A list of the selected location's transnational issues is displayed. The user is then asked if he or she wants to look up another location.  If the user types yes, the process is repeated.  If 'no' is the input, the program exits.

**Class design**

I begin the project by thinking about what types of classes I would need to create and how they would need to interact with one another.  I know I wanted a commandline interface (CLI) class to handle aspects of the program that involved collecting input and displaying output.  I also knew that a Country class would be necessary to have variables containing the country's data.  I was left to think about how to collect the transnational issues and scraper data.

A country is more than its issues, and these issues are part of a country.  I decided to make an Issues class that is part of a Country class.  There is no inheritence involved because this is not a parent/child relationship.  Now how to deal with a scraping?  Scraping is different from CLI, Country, and Issues, so I decided on a separate Scraper class.  

The last design decision involved breaking the Scraper class into subclasses:  ScraperCLI, ScraperCountry, ScraperIssues.
ScraperCLI scrapes data directly related to providing data based on commandline input.  ScraperCountry scrapes data specific to a country, and ScraperIssues scrapes data specific to a country's issues.


```
CLI                                                  
       -ScraperCLI                              
       -Country                               
                   -ScraperCountry           
                   -Issues                         
                                -ScraperIssues
```

Above diagram: Class CLI instantiates a ScraperCLI object and a Country country.  Country class instantiates ScraperCountry object and an Issues object.  The Issues Class instantiates a ScraperIssues object.  

**Outward-in development**

Once I had a general sense of class design, I started with the basic command line output and stubbed fake data to output.  Once the CLI class was working, I went one level deeper and replaced the stubbed country information with an instance of a Country class that itself was stubbed with data.  I additionally stubbed in issues data within the country object.  Once everything was runnign smoothly, I then replaced the stubbed issues data with an instance of the Issues class that iself had stubbed data.  Now I was three levels deep and just need to replace each classes' stubbed data with real data that had been scraped off the the C.I.A. website.

**Nokogiri CSS selectors failed me -- welcome, XPath.**

It was difficult to hone in on the specific information I needed.  I could obtain the data, but I'd have to iterate through what I have scraped until I found what I was looking for.  I needed a more specific way of using a 'fine-toothed saw'. I found searching by XPath online.  [A great XPath tutorial](http://zvon.org/xxl/XPathTutorial/General/examples.html).  XPath is a syntax for defining different parts of an XML document, and the expression for selecting specific nodes or node sets in the document looks very much like a file directory syntax that you use in your terminal.  [More information on XPath](https://www.w3schools.com/xml/xpath_intro.asp).


Can't wait for the next project!





