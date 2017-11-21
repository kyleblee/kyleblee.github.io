---
layout: post
title:      "My Personal EA (Sinatra Project)"
date:       2017-11-20 21:57:35 -0500
permalink:  my_personal_ea_sinatra_project
---


I just completed my Sinatra project! Aside from the Heroku deployment struggles at the end, I actually had a lot of fun building it. I’m looking forward to the upcoming *Rails* section in the curriculum, but before moving on, here’s a little glimpse of my project and the process it took to build it. 


***Coming up with an idea***

The first thing to do was come up with an idea… which can often be the toughest part. I had some initial ideas, but nothing was really sticking out to me. A pet sitter app that helped sitters keep track of owners and their various pets seemed useful! But, it was too similar to a lab we had already completed in this section of the curriculum. A French vocabulary app to keep track of phrases a user is working on might be something I could actually use! But, there wasn’t much interaction between models—it would basically just be a storage system for phrases. I had one idea related to music concerts, but decided it would be better suited for a future project (perhaps Rails or JavaScript).

Then, it hit me. A simple web-app to solve a problem I had been having for a while, in my own life. 

<iframe src="https://giphy.com/embed/VStxBrCyssRPO" width="480" height="429" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/mom-idea-VStxBrCyssRPO"></a></p>



***The Idea: My Personal EA***

The upcoming job search has me thinking a lot about the relationships I have with people, both professionally and casually. I really like meeting new friends and contacts, and I find it enjoyable (and reasonably easy) to connect with people. But, maintaining those relationships and staying in touch with people after that initial handshake can be hard and overwhelming. What was that person’s name again? Why did I want to put them in touch with so-and-so? What was their field of expertise? When was the last time we talked? What was that thing we had in common? It’s a lot to keep track of, to say the least.

So, I decided to build ***My Personal EA***! The “EA” stands for *executive assistant* and this simple web-app is meant to help with some of the same things that an executive assistant would keep track of for a busy business man. The fact is, it’s fun and easy to make contacts when networking or attending social events, but it’s hard to maintain ongoing communication with people and build relationships with each one, over time. *My Personal EA* helps people keep track of the contacts that they make, encourages them to take notes on the ongoing interactions they have with these contacts, and enables them to make plans for face-to-face time with specific people.

Think about all the amazing things that can happen when you know the right person, at the right time. It could be the friend of a friend you met at a holiday party last year, who ended up starting a new company. Or a former classmate who happens to live in Austin, TX; a city you have always wanted to visit, but never had a tour guide for. Or a previous roommate who now runs the box office at that hot new music venue in town. You get the point. 

In a nutshell, *My Personal EA* tells its users, “You know a lot of people. We’ll help you stay in touch with them.” It’s about helping the "modern-day networker" stay in touch with the important contacts in their life; whether it be for career development, for fun, or both.


***Planning***

Ok, so I had the idea. Now it was time to wrap my head around exactly what I wanted the app to do, and what models and database tables would make it happen. Some features I knew I wanted were:
* user signup/login
* a list of contacts for each user
* a list of recent interactions (sorted from most recent to least recent) that a user has had with these contacts 
* a list of plans (both past plans and future plans) that a user has made with their contacts
* creating/reading/updating/deleting (CRUD actions) for users, contacts, interactions, and plans
* the ability to logout

To do this, I would need the following database tables and corresponding models, along with some basic associations between them:
* *Users*—which should have a username, email, and password digest (since I’ll be using *bcrypt* for storing users' passwords). Users should have many contacts, have many plans, and have many interactions *through* contacts
* *Contacts*—the attributes for each contact seemed pretty intuitive: a name, email, phone number, category (are they a friend, family member, colleague, etc), their expertise (what are they good at, what’s their field of knowledge), their preferences (any preferences, really), and their location. A contact will belong to a user and will have many plans through the contact_plans join table (since it’s a *has many to has many relationship*)
* *Last Interactions*—these will represent times that a user has communicated with a contact. They’ll be simple, but helpful; with a date for the interaction and details on what occurred or what was talked about. They will belong to the contact that they correspond to
* *Plans*—finally, plans will represent actual get togethers (events? outings?) that a user might schedule with their contacts. When thinking about plans (which may be professional, or not), some attributes that seem to make sense are: a name for the plan, date, time of day, location, context (what the gathering is all about), pre-notes (things to help a user prepare for the gathering), and post-notes (which allow a user to document how it went). A plan will belong to a user and will have many contacts through the contact_plans join table (a user should be able to add numerous contacts to each plan and have numerous plans for each contact)

Alright! It seemed like I had a pretty good grasp of what I wanted the app to do and how it would be structured. Now, it was time to start building!


***Let’s start building!***

I won’t bore you with every minute detail of building the website (which may or may not have taken a whole week...). However, this would be a good opportunity to recommend some awesome resources that helped me get through this project. There were a few steps that required some research, in order to be completed successfully: 
* First, setting up a project from scratch can be somewhat intimidating. On most *Flatiron School* labs, the structure has been implemented for us, so we can focus on the particular topic at hand. **[This blog](http://blog.flatironschool.com/how-to-build-a-sinatra-web-app-in-10-steps/)** by a former student was a lifesaver when I was putting together the scaffolding of my project.
* Next, I would need to get my project hooked up to a remote repository on GitHub. It’s been awhile since my **[CLI Data Gem project](http://kyleblee.com/2017/08/02/daily_recipes_my_cli_data_gem/)** so… How do I do this, again? Oh, yeah, thanks **[GitHub](https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/)**.
* I knew before I started that I wanted to push myself beyond the requirements of the project in one way in particular: making sure it didn’t look like crap. This didn’t need to win a *Webby Award*, or anything; but I really wanted to get better at implementing custom CSS, and this seemed like a great opportunity to do so. After scouring the web for decent CSS tutorials, I found this *awesome* web series by **[DevTips](https://www.youtube.com/channel/UCyIe-61Y8C4_o-zZCtO4ETQ)** on YouTube (if you aren’t already following Travis, I highly recommend it). The **[series](https://www.youtube.com/watch?v=T6jKLsxbFg4&list=PLqGj3iMvMa4KQZUkRjfwMmTq_f1fbxerI)** is based on building a responsive website from start to finish; and, luckily for me, a lot of it focuses on making stuff look pretty with CSS. My web-app ended up looking very different than the website that Travis builds in his series (which is a good thing!), but this was a great way to get started and set up some base styles.
* And finally, I knew I would have to make this website publicly available, before I could call it a done deal. Heroku seemed like a logical choice for this, but there was a major problem: YOU CAN’T USE SQLITE3 WITH HEROKU! (dun dun dun dunnnnn, dun dun dun dunnnnn). Luckily, I wasn’t the only student who crapped their pants when they realized this, and I happened upon a conversation on the school’s Slack channel, where **[a former student’s blog post](http://lucaskisabeth.com/2017/06/24/deploying_your_sqlite3_sinatra_app_to_heroku_using_postgresql/)**—about how to deploy a SQLite3 app to Heroku using PostgreSQL—was being passed around. That’s online community for ya, at its finest.

These resources were a HUGE help in putting my Sinatra project together. Who knows… they might help you, too. 

<iframe src="https://giphy.com/embed/5wWf7GW1AzV6pF3MaVW" width="480" height="278" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/editingandlayout-the-office-high-five-5wWf7GW1AzV6pF3MaVW"></a></p>


***What’s next? (future features)***

Part of the reason I liked this idea for a web-app is that it has *a lot* of room to grow and scale. Right now, it's a fairly simplistic website, based around associations between users, contacts, plans, and interactions. I mean, let’s face it: there aren’t any intense algorithms, custom transitions / buttons, or third-party integrations at work here. But, there are SO many things that could be added on to this, to make it even more useful to a potential user. Here are just *some* of the major ones that come to mind:
* Modifying interactions to be both past *and* future (being able to schedule future interactions with reminders, pre-written messages, integration with API’s for auto-delivery, etc)
* A “today” view that has a list of all the interactions scheduled for the day; as well as any plans that might be occurring, a section for birthdays, maybe a news sidebar with stories that relate to the companies where a user’s contacts work, etc
* “Groups” or another way to represent relationships between a user’s contacts (colleagues from a certain company, freelancers, creatives, friends in the same circle, contacts in particular cities, etc)
* Allowing users to upload pictures for each user, contact, plan, and interaction (a simple add that would make the app *much* more visually pleasing)
* Making the entire site responsive and cross-browser compatible (this should probably be the very first thing that is done, when I come back to clean this up...)
* Adding signup/login options through Facebook / LinkedIn APIs
* Changing the *LastInteraction* model to a *has_many / has_many relationship* with contacts; since interactions can often involve numerous contacts at one time
* A calendar view, so a user can quickly see all of the plans and interactions scheduled for that day / week / month
* Password recovery and updating
* Adding a more graceful 404
* And, as always, diving back in and refactoring… a lot


<iframe src="https://giphy.com/embed/JIX9t2j0ZTN9S" width="480" height="480" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/JIX9t2j0ZTN9S"></a></p>


***Quick glimpse***

So that’s that (for now)! *My Personal EA*, a simple web-app to help users stay in touch with the plethora of contacts in their lives. Just in case I have peaked your interest, here is the URL where you can find my Sinatra project; along with a few screenshots of primary views. Thanks for reading!

*My Personal EA*: [https://my-personal-ea.herokuapp.com/](https://my-personal-ea.herokuapp.com/)

<a href="https://imgur.com/AH2agq1"><img src="https://i.imgur.com/AH2agq1.png" title="source: imgur.com" width="100%"/></a>

<a href="https://imgur.com/wEOVwKu"><img src="https://i.imgur.com/wEOVwKu.png" title="source: imgur.com" width="100%" /></a>

<a href="https://imgur.com/T07dSAU"><img src="https://i.imgur.com/T07dSAU.png" title="source: imgur.com" width="100%" /></a>

<a href="https://imgur.com/pBG6l1Q"><img src="https://i.imgur.com/pBG6l1Q.png" title="source: imgur.com" width="100%" /></a>

<a href="https://imgur.com/621wOQv"><img src="https://i.imgur.com/621wOQv.png" title="source: imgur.com" width="100%" /></a>

<iframe src="https://giphy.com/embed/WsKVAem02Efuw" width="480" height="455" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/dancing-happy-excited-WsKVAem02Efuw"></a></p>
