---
layout: post
title:      "PracticeBuddy (Rails Project)"
date:       2017-12-23 23:32:06 -0500
permalink:  practicebuddy_rails_project
---


Whew! After about a week of building, I’m excited to announce that I have finally completed my Rails project! Rails was a challenging but also immensely rewarding section of the course, and I am beyond excited to start on JavaScript. But before I do, how about a quick “encore” to recap the project.

***Omg... another music post!?***

As I have mentioned in a few other blog posts—such as, [Why Are You Doing This To Yourself](http://kyleblee.com/2017/05/02/why_are_you_doing_this_to_yourself/), [Code (in the Key of C Major)](http://kyleblee.com/2017/05/18/code_in_the_key_of_c_major/), and [Why Brands Should Be More Like Brands](http://redpegmarketing.com/news/why-brands-should-be-more-like-bands/)—I come from a fairly musical background. I mean, my parents don’t play any instruments, and I didn’t start playing at the ripe age of 5; but I picked up a guitar for the first time in a middle school music class and I haven’t put it down since. Hell, I was even a music school dropout before deciding that a degree in marketing was probably a better move, in terms of maintaining a “stable income”. 

While there have been ebbs-and-flows in the amount of time I have dedicated to the craft of music, it has always had an effect on the way I think and perceive the world. So, when I was brainstorming ideas for my Rails project, it was only natural that a couple ideas related to music popped up. One of them is actually something I’ve been thinking about for a while. It aims to solve a problem that most instrumentalists have had to solve on their own—often in a fairly “manual” way.

***The Problem***

Part of becoming a better musician (particularly for improvisers) is internalizing a vast and ever-growing vocabulary of musical phrases. These musical phrases—often called “licks”, “chops”, etc—are like a “bag of tricks” that a musician can pull out and use, on the spot, in various playing situations.

There are many ways to learn these phrases, such as: transcribing solos off of records, buying lick books and play alongs, swapping ideas with friends, and writing your own. However, learning the lick is really just the beginning of the process. In order to keep them fresh and usable, a player has to constantly review phrases they have learned in the past. This can become overwhelming, as serious musicians—through years of practice—will often learn hundreds upon hundreds of unique phrases, for dozens of tonal situations. I mean, that’s a lot to keep track of…

***Welcome to PracticeBuddy!***

PracticeBuddy is meant to serve as a sort of repository for a player’s constantly evolving musical vocabulary. It helps musicians keep track of all their licks—key details about each phrase, as well as dates when it should be practiced again—so that they can focus more on perfecting each lick, and worry less about remembering the ones they haven’t used in a while.

By organizing your practice routine, and making sure you never forget a lick, PracticeBuddy helps you become a better musician and improviser.

Alright, you’ve got the idea; now let’s get specific about how it works.

<iframe src="https://giphy.com/embed/5C3ES5uAlDCo" width="480" height="465" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/kangaroo-shred-5C3ES5uAlDCo"></a></p>


***Landing Page / Sign Up / Sign In***

<a href="https://imgur.com/xxfJ0n6"><img src="https://i.imgur.com/xxfJ0n6.png" title="source: imgur.com" /></a>

A signed out user will first be greeted with the landing page at `root_path`. This is a simple view that has a quick summary on *PracticeBuddy* and a navbar with links that take users to *Sign In* or *Sign Up*. Not much else to talk about here.

<a href="https://imgur.com/0MypMMK"><img src="https://i.imgur.com/0MypMMK.png" title="source: imgur.com" /></a>

The *Sign Up* page has fields for name, email, and password; which is all the information we need to instantiate a new `User` object and persist it to the database. Each of these attributes has validations defined on the `User` model:
* A *name* must be present
* An *email* must be present, unique, and adhere to typical email formatting (for which I used a `gem` called `validates_email_format_of`)
* A *password* must be present and is validated through `bcrypt` and the `has_secure_password` macro

<a href="https://imgur.com/ak1v4xD"><img src="https://i.imgur.com/ak1v4xD.png" title="source: imgur.com" /></a>

You can also sign up via Facebook, which I implemented using the following gems: `omniauth`, `omniauth-facebook`, and `dotenv`. 

<a href="https://imgur.com/QxGFkMG"><img src="https://i.imgur.com/QxGFkMG.png" title="source: imgur.com" /></a>

To sign in, the user simply enters their email—which is unique and is used to identify that user in the database—and their password, which is authenticated through `bcrypt`. For this demonstration, I’ll be using an account that I have already seeded with data, so the views are more representative of what an active user would see. 


***We’re in! (the Home view)***

<a href="https://imgur.com/KrKveZC"><img src="https://i.imgur.com/KrKveZC.png" title="source: imgur.com" /></a>

Once a user has successfully signed up / signed in, the first view they are shown is the `sessions#index` view (`home_path`). This page is meant to be a summary of that day’s practice schedule and shows the user licks which are most important for them to focus on.

To collect this information, the app uses class level scope methods to query relevant data and organize it into sections:
* **Licks of the Day**—rounds up all the `Lick` objects whose `scheduled_practice` attribute (datetime) matches the current date
* **Overdue Licks**—selects all of the `Lick` objects that were scheduled to be practiced at a past date, but haven’t yet been practiced
* **Sloppiest Licks**—this section displays licks which have the lowest integers in their `performance_rating` column. The user specifies this number, from 1 to 5, whenever they practice a lick and it’s meant to represent how well they could execute the lick that day—could they play it smoothly with confidence, or was it sloppy with mistakes? This helps them stay on top of phrases that require extra attention and practice
* **New Backing Tracks**—this final section shows them the most recently created backing tracks (created by any user), so they can glance through them and add any they like to their own licks for future practice sessions.


***Licks***

`Lick` objects, and the various CRUD actions that can be performed on them, are really the main feature of the app, since that’s what the whole concept is based around. `Lick` resources are nested under `User`, as they are specific to the individual, practicing musician.

<a href="https://imgur.com/Ld5FjPn"><img src="https://i.imgur.com/Ld5FjPn.png" title="source: imgur.com" /></a>

I will talk mostly about the `licks#index`, `licks#new`, and `licks#edit` actions/views here, since that is where most of the “action” really is.


*licks#index (URL: /users/:id/licks*
<a href="https://imgur.com/iA3MAl3"><img src="https://i.imgur.com/iA3MAl3.png" title="source: imgur.com" /></a>

Each user has a `licks#index` view where they can see all of the licks they have created. There are *filter* and *sort* select tags available for easily narrowing results to specific licks that the user is interested in. Behind the scenes, class level scope methods are querying the collection of licks that the user is looking for and then running them through a number of sort methods to prepare them for being rendered on the page. The view changes slightly depending on which sort selection has been made (for example, dates appear next to licks when *Date Last Practiced* or *Scheduled Practice Date* have been selected, or headers appear when *Tonalities* or *Artist* have been chosen, etc).

<a href="https://imgur.com/Fq3AaLp"><img src="https://i.imgur.com/Fq3AaLp.png" title="source: imgur.com" /></a>

<a href="https://imgur.com/J3hVbJY"><img src="https://i.imgur.com/J3hVbJY.png" title="source: imgur.com" /></a>


*licks#new and licks#edit (URL: /users/:id/licks/new or /users/:user_id/licks/:id/edit)*
<a href="https://imgur.com/Oi82CH5"><img src="https://i.imgur.com/Oi82CH5.png" title="source: imgur.com" /></a>

`licks#new` and `licks#edit` share the same form, which is written in a partial and rendered into each view. You will see fields (generated through the `form_for` helper) for all of the attributes that are directly associated with a lick, such as: name, bpm, current key, link, performance rating, scheduled practice, and description. 

You will also see a collection select for `Artist`, as well as collection check boxes for `Tonalities` and `BackingTracks`. `Artist`, `BackingTracks`, and `Tonalities` also have nested forms that will instantiate objects that are already associated with this particular `Lick`. These associated objects are instantiated through custom setter methods, such as: `Lick.new_backing_track=`, `Lick.new_tonalities=`, and `CustomSetters::InstanceMethods.new_artist=` (which is included in both `Lick` and `BackingTrack` models, to keep things DRY).


***BackingTracks***

<a href="https://imgur.com/TIVFQbo"><img src="https://i.imgur.com/TIVFQbo.png" title="source: imgur.com" /></a>

An additional class, called `BackingTrack`, has been included so that users can easily access online play along tracks to apply their musical phrases to. Similar to the `Lick` class, the full suite of `BackingTrack` CRUD actions are available as nested resources under `User`.

Unlike the `Lick` class, however, users can access additional `backing_track#show` and `backing_track#index` actions in routes outside of the nested resources. While licks are purposely specific to each individual user, I wanted `BackingTracks` to be more “communal”. By having nested routes and routes that aren’t nested, a user can see their personal `BackingTracks` (`/users/:id/backing_tracks`) or see all of the `BackingTracks` that have been created by any user (`/backing_tracks`). Users can also add any `BackingTrack` to their licks, for use in future practice sessions.

<a href="https://imgur.com/7tojZIh"><img src="https://i.imgur.com/7tojZIh.png" title="source: imgur.com" /></a>

There is also a nested form (well, a nested field, really) on `backing_tracks#new` and `backing_tracks#edit` for the associated `Artist` class, so that users can build `Artist` objects off of the `BackingTrack` they are currently creating / updating. This is done through the module `CustomSetters::InstanceMethods`, whose `.new_artist=` setter method is also used during mass assignment for `Lick` objects.

<a href="https://imgur.com/cknBROM"><img src="https://i.imgur.com/cknBROM.png" title="source: imgur.com" /></a>


***Errors / Flash Messages***

<a href="https://imgur.com/SxgBcTu"><img src="https://i.imgur.com/SxgBcTu.png" title="source: imgur.com" /></a>

Finally, error messages and simple error highlighting (via the `field_with_errors` class) have also been implemented so that when validation fails, the reason is clearly expressed to the user. Furthermore, flash messages are used throughout the site to communicate short notices, such as: “Lick created!”, “Backing Track deleted!”, and “Hmm, we can’t seem to find a lick like that.” Both errors and flash messages are rendered into relevant views using partials.

<a href="https://imgur.com/CPVM5sR"><img src="https://i.imgur.com/CPVM5sR.png" title="source: imgur.com" /></a>


***Now, get back to practicing! Or... JavaScript.***

So, that’s *PracticeBuddy*! A simple tool to help musicians keep track of all the musical phrases they have learned in the past and make sure that each one is ready to go for their next gig or jam session. Now, it's time to practice some chops! And after that, I think I’ll get going on the JavaScript section.

Until next time!

<iframe src="https://giphy.com/embed/PcLZLXy05qPQs" width="480" height="360" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/PcLZLXy05qPQs"></a></p>
