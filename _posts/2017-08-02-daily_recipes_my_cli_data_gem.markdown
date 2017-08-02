---
layout: post
title:  "Daily Recipes (My CLI Data Gem)"
date:   2017-08-02 14:22:06 +0000
---


## Welcome to Daily Recipes!

Hip hip hooray for completing my first code project! It’s also the first time I haven’t been simply writing code to make Learn’s tests pass—though, that’s pretty fun, too. This was my first real opportunity to work through the entire process: coming up with an idea, thinking through how it will work, making a plan, and then actually building it out. Overall, I’d say it went pretty well!

So, what did I build? Well, when I started thinking about what my big, million dollar idea was going to be, I got kind of hungry (this happens to me a lot). So, I’m in my kitchen at home, eating leftovers, when I see a massive 250lb cookbook open on the counter. Above that cookbook, in a pair of brown cabinets, the shelves are completely full with other recipe books of similar sizes and with similar headlines. I thought to myself, “Surely there is a less exhausting way to choose a recipe…” And there is, obviously; there are probably over a hundred websites that do this very thing. But, I decided to do it myself, in a slightly different and much simpler way.

Here’s the elevator pitch:
>“Searching for a recipe can be overwhelming. With seemingly hundreds of websites and cookbooks to look through, it can be hard to make a quick decision on what you want to make for a tasty personal, or family, meal. To avoid this, people frequently resort to making the same recipes, over and over again—sacrificing spontaneity for convenience. It's time for a solution.

>Welcome to Daily Recipes! Your new, handy-dandy CLI application for receiving a short, digestible list of recipes every day. At the perfect intersection of simplicity and laziness, Daily Recipes makes it easy to choose from a short menu of dishes (scraped from three popular recipe sites), get more information on any that pique your interest, and even open any of the full recipes in your preferred browser. It is simple, easy to use and saves you from having to look through countless sites just to  find one quick recipe.

> Enjoy, and happy cooking!”

Like I’ve already admitted, maybe it’s been done before; but I felt like this could be a good / useful tool, and more importantly, it was easy to envision how it would translate to object-oriented Ruby. It also provided a challenge that would require multiple layers, which was one of the primary asks for the project.

So, we have the idea… now what’s the plan? 

<iframe src="https://giphy.com/embed/xLIwD85C0z9D2" width="480" height="200" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/jennifer-lawrence-fire-american-hustle-xLIwD85C0z9D2"></a></p>

## Initial Planning

Luckily, before I started on the project, I watched Avi’s *CLI Data Gem Walkthrough* (twice...), which was immensely helpful. One of the things that Avi did, before even getting started on coding, was make a “NOTES.md” file where he explicitly wrote out what he wanted his program to do, as well as general guidelines on how it might be built. I tried doing the same thing with Daily Recipes. 

![](http://i.imgur.com/C2T3Q5V.png?1)

At this point, I felt like I had a pretty good idea of what I was building and how I was going to build it. Now it was time to roll up the sleeves, and start cooking! (please excuse the "corn"y sense of humor)


## Building it out (WARNING: it’s about to get nerdy)

The first thing I needed to do was get the project’s file structure set-up. Once again, Avi’s video really helped pave the way here. I used `bundle gem daily_recipes` to automatically generate the project files. Then, I created a remote git repository and hooked it up to my local repository, after making my first commit. 

Next, I created a "daily-recipes" file in the bin directory, which would be my primary executable, and added a shebang line. The file, for the time being, puts-ed "You ready to eat some food, yo?"—just so I could be sure it was working. Then, I created a CLI.rb file inside of my lib/daily_recipes directory for actually handling the user interaction of the program (the CLI controller) and created a class called *#start* that would greet the user and *gets* the first input (whether or not they would like to see the recipes of the day). The final *#start* method looked something like this: 


![](http://i.imgur.com/sKPtrKw.png?1)

I eventually added conditional statements inside of *#start* so that "yes" presented a menu for them to choose from and "no" said a *#goodbye* message before exiting the program. Later, I went back and added an *until* loop so that I can could give a "not sure what you meant" message for unexpected inputs and prompt the user for input again.

When I tried running the daily-recipes executable, I got an "uninitialized constant" error because my load dependencies weren't set up correctly. So, I continued to watch Avi's video and use that guidance to fix the dependencies in my program. I decided to use the lib/daily_recipes file as my environment file, similar to what he does in the video.

Next, I defined a class called "recipe" in a file called "recipe.rb" that would be responsible for creating new recipes by scraping 3 different recipe sites. It would also be responsible for returning *@@all* the recipes that had been created—so that they can be printed by the CLI class—as well as giving more detail on the specific recipe(s) that the user is interested in learning more about (by scraping individual, full recipe pages).

I had forgotten to add pry earlier, so I added it to the gemspec file and my environment file, as well. I'm still not entirely sure how Avi required dependencies in the video when he used the console file… but it seems to be working for now, so I’m just going to go with it. While adding dependencies, I added the Nokogiri and OpenURI dependencies to my gemspec and environment files so that I could start scraping.

Then, it was time to get into the nitty-gritty: scraping. For the first level, I needed three separate scraping methods (one for each recipe list page). I started with allrecipes.com as the first site to scrape from, and decided to call the method *#scrape_allrecipes_website*. First, I needed to use Nokogiri::HTML and OpenURI to define a "doc" variable that I could start searching through. To find out which CSS selectors grab the correct elements, I used Chrome's Inspect Element feature and binding.pry so that I could play around until I found the correct classes/ids. Once I found the correct selector for each recipe element (`#grid .grid-col--fixed-tiles`), I decided to include a conditional statement inside of the *#each_with_index* iterator that only creates a new recipe if the index is less than 5. I realize this was a pattern that Avi advised against using in his anti-patterns... but there are a ton of recipes on these sites and I'd rather just give 5 from each one (right from the beginning) to keep the program faster and simpler to use. It’s also one of the core differentiating aspects of my program: presenting the user with a short and digestible list of recipes—not an overwhelming laundry list. I’ll have this method return the *@@all* class variable. It won’t be used in the DailyRecipes::CLI #menu method, but it could be useful for later methods that use this method.

Once I got the correct selector for the title attribute, I ran into a problem to solve... The selector that all of the recipe grids are using is also being used by advertisements / marketing that are mixed throughout. The problem is that the h3's in these grids return an empty string (and we aren't interested in them anyways, since they aren't recipes). 

![](http://i.imgur.com/MxnfUyD.png?1)

There are two ways for me to solve this: include them anyways in the initial collection and remove them later, or figure out a way to filter them out now using a high-level enumerator like *#reject*. I decided to go with the second option, because adding them anyways and then removing them seems like extra, unnecessary work for the program. I noticed that these advertisements also have additional classes in their selectors, such as "marketing-card" or "gridad". So, I used the *#include?* method inside *#reject* to determine whether or not the selectors contain "marketing-card" or "gridad" in their `attr("class")`. That worked!

Now that I removed the recipe articles that shouldn't have been included, and the recipe titles were being set correctly, it was time to move on to the URL attributes of recipe objects. For the initial list, I decided I would just get the title of the recipe and its URL (which I could use later to scrape the rest of the data for that recipe or take the user to that webpage).

The URL attribute was easy enough to complete, by using "https://www.allrecipes.com" and then interpolating with the selector `recipe.css("a").attr("href").value` for each recipe element.

![](http://i.imgur.com/vGECrko.png?1)

One down, three to go! To spare you some of the painstaking detail, I’ll just say that my process for the other two was extremely similar to the one I just completed (using different selectors for each site). Let's fast forward through this part.

<iframe src="https://giphy.com/embed/l3q2IYN87QjIg51kc" width="250" height="250" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/creativemornings-time-l3q2IYN87QjIg51kc"></a></p>

Now that all of the recipe lists were being scraped and recipe objects were being initialized successfully, it was time to turn these into a menu that the user can see and choose from. I decided that this should be the *DailyRecipes::CLI.menu* method's job, once it received *@@all* from recipe.rb. It was fairly easy, using *#each_with_index*, to *puts* each recipe title. Later, however, I split up the functionality more by having a separate method called *#print_menu* handle the actual printing (the reason for that split is explained later).

![](http://i.imgur.com/jiDmfOB.png?1)

![](http://i.imgur.com/nOIm7bF.png?1)

As you can see at the bottom of that screenshot, I also added the next level of user interaction. It will involve allowing the user to input a recipe's number (from the ordered list) to get a little more information about that recipe (a description and total prep / cook time). To do this, it needs to take the user's input, have some new scraping methods for scraping full recipe pages, determine which new scraper to use by looking at the *@url* attribute of that recipe, scrape that *@url* using the correct scraper method, add those new attributes to the object, and then return the new, more detailed object. Easy, right? ![](http://i.imgur.com/z6XeMn0.png?1)

Well, actually, when I was thinking about how I was going to build this next step, I immediately uncovered a problem: recipe.com aggregates it's recipes from a number of **other** recipe sites and simply presents them together on their recipe list page (sort of like what I'm doing with Daily Recipes). This means that I'd need a different scraper method for **each** site that they draw from, a way to determine which method to use, and even *then* it would be extremely error prone. So... looks like I'll need to go back, replace recipes.com with a site that doesn't send the user to other recipe sites, and build the scraper method again... great. Delish.com seems to keep everything "in-house"—so, let's go with that. Please hold.

<iframe src="https://giphy.com/embed/l3q2IYN87QjIg51kc" width="250" height="250" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/creativemornings-time-l3q2IYN87QjIg51kc"></a></p>

Now that I incorporated delish.com instead, it was time to get going on what will happen after the user makes a choice. I decided to incorporate a conditional statement, using *#include?* inside the condition, to determine which details scraper to use (based off of the *@url* instance variable for each recipe object). Then, the program will use that scraper method to add the extra information to the recipe instance and return that object to  be printed. Makes sense, let’s do it.

And… just when you think it all makes sense… BAM! Another roadblock. When I started to use Nokogiri on the first details scraper method, I ran into an error I haven’t seen before on Learn. I went to grab the HTML using Nokogiri and OpenURI, and I got the following: 
`'open_loop': redirection forbidden: https://allrecipes.com/recipe/258306/nourish-refresh-smoothie/ -> http://allrecipes.com/recipe/258306/nourish-refresh-smoothie/ (RuntimeError)`

Time to put on the problem solving cap. I entered this error into Google and tried some of the fixes I was seeing on Stack Overflow, but quickly realized I was in a little over my head. However, I noticed that on a Treehouse forum, someone who was having a similar problem simply used *#gsub* to switch "http" to "https". Looking at my code, it seemed that I was having the OPPOSITE problem, so I tried using *#gsub* to replace "https" with "http" and VOILA! It worked! Not sure I really understand what is going on here, but I decided to move on for now. At least I wasn't forced to use an entirely different recipe site (again...). Below is the fix on Treehouse, as well as my solution:

![](http://i.imgur.com/Ntdfy2F.png?1)

![](http://i.imgur.com/Q8WFJfb.png?1)

Ok, making progress again. The recipe.description wasn't too bad for allrecipes.com. But, then I hit another small obstacle when I was getting ready to look at how to scrap the total prep / cook times... For whatever reason, the times are presented with the number of minutes or hours (the actual integer itself) in it's own `<span>` class, and then the hours or minutes abbreviation outside of that `<span>`. Furthermore, not all recipes take over an hour, so I would have to make the distinction as to whether or not the interpolation should include both units of time, or just minutes. Fun…? No, not really.

![](http://i.imgur.com/XIt9nY0.png?1)

After struggling in pry to try and find the right selector, I decided to scrape a list of ingredients instead. This will still be helpful for the user, and it should be a lot easier to iterate through. If I wanted to collect the prep / total cook times, not only would I have to implement conditional statements to distinguish between times under and over an hour, but even just getting that far required **more** conditional statements because there were numerous `<span>`s with `"itemprop="` attributes... so I would need to find the one with a value of `"totalTime"`—which would be a total pain.

Looking at the three sites I was pulling from, ingredients definitely seemed like an easier bet. The way they are set up in each full recipe page should allow for easy iteration to push them into a collection and then print them for the user. I think an ingredients list is still a helpful feature, because the user can make recipe decisions based on ingredients they currently have or will need to buy. So... I’m going for it. :)

On allrecipes.com, scraping the ingredients into an array was pretty easy. Just needed to remove an "Add all ingredients to list" button that had the same class as the ingredients. This button also had an additional class that I could use to remove it, using the *unless* keyword with the *#attr* method to look at the class of each ingredient element.

![](http://i.imgur.com/szA86yl.png?1)

Now I’ll apply the same logic to the other two sites. Please hold.

<iframe src="https://giphy.com/embed/l3q2IYN87QjIg51kc" width="250" height="250" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/creativemornings-time-l3q2IYN87QjIg51kc"></a></p>

Alright, at this point, we had built the additional scraping methods to get more information for the recipe that the user is interested in, and the class method *#recipe_description_card* was returning that more detailed recipe instance. We could now use that returned object in the CLI to print out the information for the user and prompt them for the next, final layer of interaction: being taken to the full recipe page itself in their browser, seeing the menu again, or exiting the program.

To do this, the “code I wish I had" would be a method that took in the *detailed_recipe* as an argument, called a separate method called *#print_recipe* to print that detailed_recipe, and gave the user another set of options. Printing the recipe was easy using the recipe.title, recipe.description, and iterating through the recipe.ingredients. I decided to place the final layer of the user flow (the choice to navigate to the recipe's page for full directions, go back to the list, or exit the program) in it's own method called *#full_directions?*.

![](http://i.imgur.com/q5Aqdt0.png?1)


I started to build *#full_directions?* in a very similar way to *#start*, in the sense that it prompts the user for a response and then starts up an until loop which doesn't break until the user has typed 'more' (to be taken to the full recipe in-browser), 'menu' (to see the menu again), or 'exit' (to exit the program). I got it up and running, but when I went to test it, I noticed that now, if the user types 'exit' in the previous input interaction (after seeing the daily list of recipes), they are shown *#goodbye* but are then asked if they would like to see the full recipe... 

![](http://i.imgur.com/ctT4Flx.png?1)

This is because in *#start*, *#full_directions?* is running after *#choice* runs, regardless of what the user entered. I fixed this by using the return value of *#choice* to determine whether *#full_directions?* should run or not. If the user has entered a recipe they are interested in, then a local variable *chosen_recipe* will be set to that recipe so I can use it in *#full_directions?* by passing it in as an argument. However, if the user types "exit", `nil` will be returned and I'll use an *unless* keyword to avoid *#full_directions?*.

Another problem popped up when the user wanted to see the menu again. Presenting the menu a second or third time caused duplicate menu items, because in *#menu* the program first scrapes the sites and creates recipe objects before printing the menu, every time. This meant that if the menu had already been shown, and the user entered 'menu', the program was creating all of the recipe objects again, adding them to the *@@all* class variable, and then printing all of the recipes stored in that class variable. This was a simple fix: I just removed the print functionality from *#menu* and placed it in *#print_menu*. Now, both *#menu* and *#full_directions?* can simply call *#print_menu* and avoid any duplicating behavior.

Then, it was time for the final step: taking the user to the recipe's webpage if 'more' is entered. To open the full recipe webpage, I used the *#open* method built into Ruby and simply interpolated the chosen recipe's url attribute into *#open*’s argument. After the user enters 'more' and is taken to the webpage, the program prints out a prompt to either see the ‘menu’ again or enter ‘exit’. I then set the local variable containing their choice to `nil` so that the *until* loop iterates again. I like this implementation because I can just use the logic I have already built (within the *until* loop).

![](http://i.imgur.com/IyHMaBX.png?1)

## Bon appetit!

And with that, my daily-recipes is done!! I'm going to go back through to add comments, which will make things more clear for other developers, and do any immediate refactoring that I see!

<iframe src="https://giphy.com/embed/11sBLVxNs7v6WA" width="480" height="217" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/cheer-cheering-11sBLVxNs7v6WA"></a></p>

![](http://i.imgur.com/VQKlAjd.png?1)
