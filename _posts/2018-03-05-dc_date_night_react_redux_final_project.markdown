---
layout: post
title:      "DC Date Night—React / Redux Final Project"
date:       2018-03-05 15:38:31 -0500
permalink:  dc_date_night_react_redux_final_project
---


Woah… it’s time for my *final* Flatiron School project! It’s definitely bittersweet, but overall I am excited and happy to be graduating! I'm also excited because, in my humble opinion, I saved the best project for last. This is something I plan on continuing to build out, moving forward; but it’s at a good initial stopping point right now, so let’s get into it!

***Idea***

First, a little bit about the idea. DC Date Night is really centered around this feeling that: *Dating can be pretty stressful.* But, when you’re the person who takes on *planning* the date, *it can be even more stressful.* Whether you’re making a first impression, or planning something special for a longtime significant other, you want the date to be memorable, fun, and unique—all while having a feeling of spontaneity. Yikes…

DC Date Night aims to relieve that stress, by planning the date for you. It basically says, “Hey, you just worry about which outfit to wear. I'll figure out where you should go.” It does this in two ways, both of which are organized by the different neighborhoods throughout the District of Columbia.
* *Curated Dates*—these are dates put together by local DC experts. These experts might be food critics, mixologists, people in the know about art exhibits and museums, etc. They curate a phenomenal date and our users go along for the ride.
* *Generating Custom Dates*—the second way to plan a date is by generating something custom. The user simply selects the neighborhood they would like to be in, chooses a few activities they want to do throughout the date, and our app generates a custom date, just for them.

Voila, your date is planned. Throw on a nice pair of shoes and go get ‘em!

Now that you get the gist of the problem I am trying to solve, let’s walk through some of the initial views and features I have put together, so far. This web app uses React / Redux on the front end, with a Rails API back end. I’ll be focusing on React and Redux for this post!

<iframe src="https://giphy.com/embed/ikXcqqlSNH2Mw" width="480" height="358" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/ikXcqqlSNH2Mw"></a></p>

<br>


***Homepage***

![](https://i.imgur.com/oDp8z0q.png)

The homepage is constructed with a stateful container component, called `Homepage` (duh!). The `return` statement in its `render` method is composed of two stateless functional components: `DateList` (which is passed `dates` as `props`) and `QuickAbout`. 

I use the `componentDidMount` lifecycle method on `Homepage` to fire the `fetchDates` function in `dateActions.js`, once the component has mounted. This is the action that is responsible for asynchronously fetching dates throughout the application and it takes two parameters: the neighborhood that the user wants to see dates for (or just a blank string, to receive all dates) and the max number of dates it should receive in the response. In the case of the homepage, we set this cap to 20.

```
// Homepage.js
componentDidMount() {
    if (this.state.neighborhood || this.state.neighborhood === "") {
      this.props.fetchDates(this.state.neighborhood, 20);
    }
  }
```

```
// dateActions.js
export function fetchDates(neighborhood, cap) {
  let headers = new Headers();
  headers.append('Content-Type', 'application/json');

  return (dispatch) => {
    const reqNeighborhood = JSON.stringify({
      neighborhood: neighborhood,
      cap: cap
    });

    return fetch('/date_entries/browse', {
      method: 'POST',
      headers: headers,
      body: reqNeighborhood
    })
      .then(response => response.json())
      .then(responseJSON => {
        dispatch({type: 'FETCH_DATES', payload: responseJSON})
      })
  }
}
```

The homepage also has the About section, which is rendered using the presentational component, `QuickAbout`. I implemented the `scrollIntoView` method on the `window` object to gracefully bring the user to this information about the site, when they select “About” from the nav bar.

```
// Homepage.js
render() {
    const { hash } = window.location;

    if (hash === "#about") {
      setTimeout(() => {
        const element = document.getElementById('quick-about');
        const datesLoaded = document.getElementsByClassName('date-card').length > 0;
        if (datesLoaded) {
          element.scrollIntoView({behavior: 'smooth', block: "start", inline: "nearest"});
        }
      }, 250);
    } else {
      window.scrollTo(0,0);
    }
```

![](https://i.imgur.com/jFGimmd.png)

<br>

***Browse***

![](https://i.imgur.com/uyTcPCn.png)

Next is the browse feature. This is the view that is responsible for displaying all of the curated date options, sorted by neighborhood. Available neighborhoods are collected via the `collectNeighborhoodOptions` function inside `dateActions`, which dispatches an action of `type: ‘COLLECT_NEIGHBORHOOD_OPTIONS’` and a `payload` of the parsed response that was received from the Rails API. In `datesReducer`, the corresponding case statement updates the `options` object in the `store`. These options are then rendered via the `NeighborhoodSelect` presentational component in `BrowseDates`. The `componentDidMount` method helps kick this process off, each time the `BrowseDates` component has successfully mounted.

```
// dateActions.js
export function collectNeighborhoodOptions() {
  return (dispatch) => {
    return fetch('/neighborhoods/options')
      .then(response => response.json())
      .then(responseJSON => {
        let options = responseJSON.map(n => n.name);
        dispatch({type: 'COLLECT_NEIGHBORHOOD_OPTIONS', payload: options})
      })
  }
}
```

Now that a neighborhood is selected, dates for that neighborhood are fetched asynchronously via the `fetchDates` function (which was mentioned earlier). We set the cap parameter to `undefined` in this case, in order to receive ALL of the relevant dates (with no cap). This updates the `curatedDates` array in `store` via `FETCH_DATES` in `datesReducer`’s switch statement.

```
// datesReducer.js
export default function datesReducer(state = {
  curatedDates: [],
  customDate: undefined,
  options: {},
  editCuratedDate: undefined
}, action) {
  switch(action.type) {
    case 'FETCH_DATES':
      return Object.assign({}, state, {curatedDates: action.payload})
    case 'COLLECT_NEIGHBORHOOD_OPTIONS':
      return Object.assign({}, state, {
        options: Object.assign({}, state.options, {
          neighborhoods: action.payload
        })
      });
    case 'COLLECT_CATEGORY_OPTIONS':
      return Object.assign({}, state, {
        options: Object.assign({}, state.options, {
          categories: action.payload
        })
      });
    case 'STORE_DATE':
      return Object.assign({} , state, {
        customDate: action.payload
      });
    case 'RESET_CUSTOM_DATE':
      return Object.assign({}, state, {
        customDate: undefined
      })
    case 'SET_EDITCURATEDDATE':
      return Object.assign({}, state, {
        editCuratedDate: action.payload
      })
    case 'CLEAR_EDITCURATEDDATE':
      return Object.assign({}, state, {
        editCuratedDate: undefined
      })
    default:
      return state;
  }
}
```

Finally, these dates are then rendered in `BrowseDates` via the `DateList` presentational component (which was also used in the `Homepage` container component). Both `BrowseDates` and `Homepage` access the dates via `this.props`, which is made possible by the `mapStateToProps` function, as well as `connect` from the `react-redux` module. Similarly, the actions that are being called in each component are made available via the `mapDispatchToProps` function.

```
// BrowseDates.js
const mapDispatchToProps = (dispatch) => {
  return bindActionCreators({
    collectNeighborhoodOptions: collectNeighborhoodOptions,
    fetchDates: fetchDates
  }, dispatch);
}

const mapStateToProps = (state) => {
  return {
    neighborhoods: state.dates.options.neighborhoods,
    dates: state.dates.curatedDates
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(BrowseDates);
```

Right now it is just this one date (seen in the screenshot above), since this is the development site; but, ideally, there would be a lot more options available. One of the features I plan on implementing when there are more options is pagination, to make the number of dates on each page less overwhelming by capping it at 15 or so.

If no curated dates are available for a particular neighborhood, a message appears telling the user so and suggesting that they try to generate a custom date for that neighborhood, instead. This is done by placing the informational message in the `errors` key of `BrowseDate`’s component state, and then passing that value from the state into the `Errors` component via the `errors` prop. This `Errors` component is used to conditionally surface error messages throughout the application when these messages are present in the containing components state.

<br>

***Curated Date Show View***

![](https://i.imgur.com/3vKcPLD.png)

![](https://i.imgur.com/oeWn0Ld.png)

![](https://i.imgur.com/Uho6Glz.png)

If a user clicks on a date in the Homepage or Browse views, they are routed to the `ShowDate` component via `React Router`. This view is really the user’s itinerary for the date. It tells them the neighborhood it is in, gives some detail about the date as a whole, shows them an image of the neighborhood, and then goes into detail on each spot that the couple will experience throughout the date.

`ShowDate` iterates through the spots of the selected date and generates a `SpotCard` component for each one, passing in the props that they need to render the correct information. The information needed for each spot includes: the name of the spot, an image for the spot (or a default image, made possible via the super handy `Paperclip` gem), and a description of why that particular spot was chosen for this curated date.

This example date (see screenshots above) is in the Shaw neighborhood of DC and is oriented towards beer lovers. Couples will begin at *Right Proper Brewing Company*, then get outside and head to *Dacha Beer Garden*, and finally end up at *Flash* for some late night electronic music.

<br>

***Curate***

![](https://i.imgur.com/e7mD9Vb.png)

If the user selects “Edit Date” on a date’s show view, or they select “Curate” from the nav bar, they will be routed to the `CuratedDateForm` component. This is the form used for both creating and updating curated dates.

The form has a number of input fields used for collecting information about the new or existing date. This includes: the date’s title, a description for the date, the neighborhood it is in, a cover photo, and the ability to add as many spots as needed. Each `SpotForm` component (used inside the `CuratedDateForm` to collect information on each spot) has fields for: the name of the spot, the category the spot is in, a description, and a photo.

These fields, as with all of the text inputs throughout the app, are controlled form inputs. This means that their values are directly attached to their component’s state and are they updated using a host of `onChange` callback functions, such as: `updateInput`, `updateSpotAttributes`, `updateCoverPhoto`, `updateSpotPhoto`, `addAdditionalSpot`, etc.

In cases where there are stateless functional components handling the input of specific information inside larger container components (`SpotForm` inside `CuratedDateForm` being a good example), these functions used in event handlers have been passed in as props and use the `bind` keyword to ensure that `this` continues to reference the parent container component.

```
// CuratedDateForm.js
generateSpotForms() {
    const categories = this.props.options.categories;

    const spotForms = this.state.spots.map((spot, index) => {
      let uniquePhotoVar = `spotPhotoField${index}`;
      return (
       <SpotForm
         key={index}
         title={spot.title}
         description={spot.description}
         updateSpotAttributes={this.updateSpotAttributes.bind(this)}
         categories={categories}
         index={index}
         selectedCategory={spot.category}
         deleteSpot={this.deleteSpot.bind(this)}
         updateSpotPhoto={this.updateSpotPhoto.bind(this)}
         photoRef={(field => (this[uniquePhotoVar] = field)).bind(this)}/>
      )
    })

    return spotForms;
  }
```

If a user wishes to edit an existing date, the same form is presented but it is auto-filled with the existing information for that date. This is done by firing the `fetchExistingDate` function in `dateActions` and setting the initial state of the `CuratedDateForm` component to the response (by updating and accessing `editCuratedDate` in `store`). Since `CuratedDateForm`'s fields are controlled, they will immediately reflect the values that are set in its initial state.

A word on next steps for this particular feature. Obviously, this form is something that should only be accessible to logged in users; and, at first, it will probably be further limited to “expert” users who have the authorization to curate dates. So, authentication and authorization are features that I will need to add in the very near future. Eventually, the ability to curate dates may be opened to all users who have an account, to make the app more interactive.

<br>


***Generate***

![](https://i.imgur.com/xfidCU0.png)

If a user doesn’t find anything that they like in curated dates, or if they just want more control over which activities they do throughout the date, they can have a custom date generated, instead. This is accomplished using the `GenerateDateForm` component.

First, they select a neighborhood from the `NeighborhoodSelect` component. Once a neighborhood has been selected, `collectCategoryOptions` fetches activity options that are available for that specific neighborhood. This is accomplished in the callback of `setState` inside `updateNeighborhood`, so that we can ensure the neigborhood has been set in `GenerateDateForm`’s state and can be included in the request for activities.
```
// GenerateDateForm.js
updateNeighborhood = event => {
    this.setState({
      neighborhood: event.target.value,
      activities: []
    }, () => {
      if (this.state.neighborhood !== "") {
        this.props.collectCategoryOptions({neighborhood: this.state.neighborhood})
      };
    });
  }
	
	// dateActions.js
export function collectCategoryOptions(neighborhood) {
  return (dispatch) => {
    let headers = new Headers();
    headers.append('Content-Type', 'application/json');

    return fetch('/categories/options', {
      method: "POST",
      headers: headers,
      body: JSON.stringify(neighborhood)
    })
      .then(response => response.json())
      .then(responseJSON => {
        let options = responseJSON.map(n => n.name);
        dispatch({type: 'COLLECT_CATEGORY_OPTIONS', payload: options});
      })
  }
}
```

The user can then select as many activities (called `categories` on the back end of the app) as they wish before clicking “Plan My Date”. This button is only available if they select a neighborhood and at least one activity, as that is how appropriate spots are queried from the database.

Once a user has generated their custom date, they are shown the itinerary that they will embark on! The spots that are chosen for the date are in the neighborhood that the user has selected and belong to one of the categories that the user has indicated they are interested in. If numerous spot options are available for one of the neighborhood and activity combinations, a spot is selected at random—in the future, this may be further customized so that the most relevant spots are surfaced based on each user's preferences. There is also a button at the bottom of this view that allows the user to go back, in case they want to roll the dice again.

![](https://i.imgur.com/4h0mYgs.png)

<br>


***Next Steps***

Part of the reason I loved the idea of DC Date Night is because of all the possibilities for expanding functionality moving forward. This is a project that I intend to keep building and improving on. One obvious feature that I plan on implementing soon is the ability to tap into third party APIs for the generation of custom dates. Currently, all of the date spots in the application are manually entered into the database; but I’m excited to start pulling from other relevant platforms, such as: Foursquare for food recommendations, Ticketmaster for live music, Eventbrite for various events and city happenings—the possibilities are endless.

As I mentioned before, I am also excited to roll my own authentication. In doing so, we can give users the ability to create and share their own dates, that they think other users would enjoy. This will make the platform more interactive and social.

Since I come from a strategy background, the marketing possibilities also jump out at me. DC Date Night could forge partnerships with various local influencers, whether they be social media personalities, food writers, cultural trendsetters, etc. These influential figures could share their knowledge and create their own curated dates for our users (and their followers) to try. Similarly, we could implement "sponsored curated dates," so local restaurants, cocktail bars, music venues, and other partners could promote their brand in a unique and relevant way.

Finally, there are a lot of ways I could dive into new tools, frameworks, and integrations, as I continue  to build out this product. Integrating Google Maps to show the spots geographically, learning React Native to bring this idea to mobile devices in a native way, using OAuth so users can login via their Facebook (or even dating app) account. There is a lot of work to be done, and so many directions it could go in.

<br>


***Thanks!***

So, that’s DC Date Night, up to this point! A simple web app that aims to make dating a little less stressful and a lot more fun—for first dates and 25th year anniversaries alike. Keep checking in to watch it grow and, if you’re ever in the DC area, give one of our dates a try! Just remember, if there is food involved, take small bites.

<iframe src="https://giphy.com/embed/jU2ZYjA8ngdKU" width="480" height="270" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/jU2ZYjA8ngdKU"></a></p>




