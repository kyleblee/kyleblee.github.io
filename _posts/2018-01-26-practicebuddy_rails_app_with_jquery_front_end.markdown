---
layout: post
title:      "PracticeBuddy—Rails App with jQuery Front End"
date:       2018-01-26 12:25:11 -0500
permalink:  practicebuddy_rails_app_with_jquery_front_end
---


***Encore, PracticeBuddy! Encore!***

For my Rails project, before entering JavaScript land, I built something called *PracticeBuddy*—a simple web-app that helps musicians track and maintain an ever growing musical vocabulary. Here’s a brief reintroduction, for context:

As musicians (specifically, improvisers) practice and develop over time, they internalize hundreds upon hundreds of musical phrases (licks, chops, whatever you want to call them). These phrases serve as a “bag of tricks,” that the performer can use in various playing situations. The problem is… it can be really difficult to remember and keep track of all of them, in order to keep them fresh. The idea behind *PracticeBuddy* is to serve as a kind of “repository of phrases,” remembering every lick that a musician has ever learned and helping them practice these licks in an efficient manner. This way, the performer can focus on the important thing: keeping their arsenal clean, up to tempo, and ready to use at any moment.

Ok, you get the idea, back to business. I’m not going to go through every feature/view of the app, since I already introduced it in my [previous blog post](http://kyleblee.com/practicebuddy_rails_project). Instead, I’m going to talk about where I have implemented AJAX and jQuery, to fulfill the *Rails App with jQuery Front End* project requirements and make the website more interactive and efficient. 

Hit it.

<iframe src="https://giphy.com/embed/NU8tcjnPaODTy" width="480" height="282" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/smooth-jazz-NU8tcjnPaODTy"></a></p>


***Licks#Index***

<a href="https://imgur.com/CdOeA77"><img src="https://i.imgur.com/CdOeA77.png" title="source: imgur.com" /></a>

The `Licks` resource is really the core of the app, since *PracticeBuddy* is based around musical phrases (AKA licks). So, I thought it would make sense to implement the AJAX and JSON functionality there, first.

The `licks#index` view now makes an AJAX GET request for JSON to the `licks#index` action. This action now utilizes `respond_to` in order to serve as a JSON API (as I am doing here) or to implicitly render the correlated html.erb template, depending on what the request is asking for.

*licks.js*
```
function renderLicks() {
  // pull user ID from the url for JSON request
  const userId = window.location.href.match(/\/users\/\d\/licks/)[0].match(/\d/)[0];
  const filterAndSortParams = {
    filter: $('select#filter')[0].value,
    sort: $('select#sort')[0].value
  }

  //send ajax request and delegates to appropriate callback function to display licks
  $.getJSON(`/users/${userId}/licks`, filterAndSortParams, displayCallback(filterAndSortParams));
}
```

*licks_controller.rb*
```
def index
    @user = current_user
    @licks = Lick.filter_and_sort_licks(@user, params)

    respond_to do |format|
      format.html { render :index }
      format.json { render json: @licks }
    end
  end
```

For requests that specify a `dataType` of JSON, this action uses an ultra-convenient Active Model Serializer to format the data queried from the database as a JSON response. It’s amazing just how easy `active_model_serializers` makes this...

*lick_serializer.rb*
```
class LickSerializer < ActiveModel::Serializer
  attributes :id, :name, :user_id, :last_practiced, :scheduled_practice, :bpm, :current_key, :performance_rating, :description
  has_many :tonalities, through: :lick_tonalities
  has_many :backing_tracks, through: :backing_track_licks
  has_many :notes
end
```

 The data is then rendered in the browser using jQuery and the *Handlebars* templating engine (which I much prefer over *Lodash*, for templating).
 
Probably the trickiest part of refactoring this to use AJAX / jQuery was the Sort and Filter functionality, which is also now available without page reloads. Luckily, my `Lick` model is still doing the heavy lifting, in terms of querying the database; but I had to reimplement all of the conditional formatting and templating that each option requires (such as tonality or artist headers, dates for last practiced or scheduled practice, etc).

<a href="https://imgur.com/1Hx5uRY"><img src="https://i.imgur.com/1Hx5uRY.png" title="source: imgur.com" /></a>
<a href="https://imgur.com/DJSevps"><img src="https://i.imgur.com/DJSevps.png" title="source: imgur.com" /></a>


***Licks#show***

<a href="https://imgur.com/T4cVo4O"><img src="https://i.imgur.com/T4cVo4O.png" title="source: imgur.com" /></a>

When a user clicks a lick's link on the `licks#index` view, in order to see a specific `lick#show` page, that content is now loaded via AJAX. It does this using jQuery, Handlebars, and a series of callback functions to replace DOM elements in the original `licks#index` view with `licks#show` elements, and vice versa.  Essentially, the user can now browse through their full library of licks without needing to do a full-fledged page reload!

*licks.js* (hijacks click events on lick list items)
```
function attachLickListeners() {
  //attach click event handler to lick list items to trigger their show view with AJAX / Handlebars
  $('a.lick-list-items').on('click',function(e) {
    e.preventDefault();
    const id = parseInt(this["dataset"]["id"]);
    const user_id = parseInt(this["dataset"]["user_id"]);
    showLick(id, user_id);
  })
}
```

*licks.js* (send the GET request for additional data via AJAX, remove `licks#Index` elements, and generate `licks#show` elements)
```
function showLick(id, user_id) {
  //display lick#show view by sending AJAX GET request, removing lick#index elements, and adding lick#show elements
  $.getJSON(`/users/${user_id}/licks/${id}`, function(data) {

    const lick = new Lick(data);

    //update header directly by replacing with name of lick
    $('#licks-header').html(lick.name);

    //piece together show view with series of DOM manipulations and Handlebars templates
    lick.displayBasicLickInfo();
    lick.displayLickTonalities();
    lick.displayLickBackingTracks();
    lick.displayLickShowOptions();
    lick.displayNotes();
  });
}
```

A user can still access individual `lick#show` pages (at their specific URL) the way they used to—with an HTML request to the `licks#show` action—from other parts of the site. This is still the default when navigating from the homepage where AJAX hasn’t been implemented yet, for example.


***Notes#index and Notes#new***

I have also added something new to the project: *Notes*. This feature is meant to be a tool that practicing musicians can use to leave themselves bits of advice, track changes, or perhaps give themselves feedback over time. *Notes* are simple, and they provide most of their value by adding additional information / interactivity to the `licks#show` views.

*note.rb*
```
class Note < ApplicationRecord
  belongs_to :user
  belongs_to :lick

  validates :content, presence: true

  before_validation :nil_if_blank

  NULL_ATTRS = %w( content )

  def nil_if_blank
    NULL_ATTRS.each {|attr| self[attr] = nil if self[attr].blank?}
  end
end
```

<a href="https://imgur.com/ONI8JNo"><img src="https://i.imgur.com/ONI8JNo.png" title="source: imgur.com" /></a>

On pages where `Lick` data is being requested via JSON, the corresponding `Note` data is included in the JSON response, as it is an association included in the `LickSerializer`. This is also true about tonalities and backing tracks, by the way.

*lick_serializer.rb*
```
class LickSerializer < ActiveModel::Serializer
  attributes :id, :name, :user_id, :last_practiced, :scheduled_practice, :bpm, :current_key, :performance_rating, :description
  has_many :tonalities, through: :lick_tonalities
  has_many :backing_tracks, through: :backing_track_licks
  has_many :notes
end
```

When a user wants to create a new note, they simply add some text to the textarea and submit it with the “Add note” button, which is hijacked with a click event handler. This click event uses `.serialize()` on the form (which is 100% magical) and sends an AJAX POST request to the `notes#create` action. If  successful, the response is the newly created note, represented in JSON format. This response is generated using an Active Model Serializer for notes, called `NoteSerializer`.

<a href="https://imgur.com/0f0wwdn"><img src="https://i.imgur.com/0f0wwdn.png" title="source: imgur.com" /></a>

*notes_controller.rb*
```
def create
    @lick = Lick.find_by(id: params["lick_id"].to_i)
    @note = @lick.notes.build(note_params)
    if @note.save
      render json: @note, status: 201
    else
      render json: @note, status: 400
    end
  end
```

*note_serializer.rb*
```
class NoteSerializer < ActiveModel::Serializer
  attributes :id, :content, :user_id, :lick_id, :created_at
  belongs_to :user
end
```

The JSON response is then put into a presentational format and prepended to the notes list for that specific lick. This way, the user can immediately see their new note, without a page reload.

<a href="https://imgur.com/8UJL3Vt"><img src="https://i.imgur.com/8UJL3Vt.png" title="source: imgur.com" /></a>


***Lick and Note Object Models***

Another requirement of this project was to utilize JavaScript OOP patterns, specifically by turning JSON responses into object models and attaching commonly used functionality to their prototypes, as methods. So, once I had the jQuery / AJAX working correctly, I went back through and refactored things to utilize this pattern.

Both *Licks* and *Notes* now have their own constructor function (which utilizes a custom “mass assignment” pattern), along with a few useful methods. 

*licks.js*
```
let Lick = function(attributes) {
  const keys = Object.keys(attributes);
  for (let key of keys) { this[key] = attributes[key] }
}
```

Most of the methods defined on each prototype have to do with formatting pieces of information (such as dates) and rendering templates in the browser. 

*licks.js*
```
Lick.prototype.formatDate = function(desiredDateKey) {
  const date = new Date(this[desiredDateKey]);
  let dateInfo = `${date.getUTCMonth() + 1}/${date.getUTCDate()}/${date.getUTCFullYear()}`;
  return dateInfo;
}
```

*notes.js*
```
Note.prototype.showNewNote = function() {
  $('form#new-note-form textarea').val("");
  $('div#notes-errors').empty();
  let newLickHTML = `<p><strong>${this.formatCreatedAtDate()}</strong></p><p class="notes-p-tags">${this.content}</p>`
  $('div#notes-list').prepend(newLickHTML);
}
```


***On to React!***

So, that is *PracticeBuddy* up to this point! It definitely could use some CSS beautification in the near future; but the app is certainly more interactive than it was before! And music is all about interaction, right? 

<iframe src="https://giphy.com/embed/12ezUIXyrnih9e" width="480" height="480" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/girls-teen-iggy-12ezUIXyrnih9e"></a></p>

